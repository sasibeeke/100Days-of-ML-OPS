The xFusionCorp Industries ML platform team is tasked with deploying fraud-detection models utilizing a compact PyTorch network. It is essential that the training script operates seamlessly on any available accelerator,
whether that be CUDA GPUs in the production cluster or standard CPUs on the lab's nodes. This ensures uniform functionality across all platforms.

Currently, a preliminary version of the trainer is available at /root/code/fraud-detection/src/models/train_pytorch.py. However, this script is not device-aware; it assumes the presence of CUDA, resulting in 
failures upon the first tensor operation on incompatible hardware. Additionally, the device parameter logged to MLflow is hardcoded, leading to inaccuracies in reporting.

Your objective is to enhance the trainer by implementing device awareness, enabling it to accurately reflect the utilized device in the MLflow logs. Furthermore, you are required to incorporate per-epoch checkpointing
to facilitate resuming long training sessions.<br>

1) The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm—the dashboard loads with an empty pytorch-training experiment. PyTorch (CPU build)
is baked into the lab image; import torch works out of the box. The host does not expose a GPU (torch.cuda.is_available() returns False).

2) The project layout under /root/code/fraud-detection/:<br>

    * data/train.csv – The same 200-row synthetic binary-classification dataset the rest of the Training section uses.
    * src/models/train_pytorch.py – The trainer scaffold. The two-layer feedforward network, the optimiser, the loss function, the MLflow experiment setup, and the model-persistence call to models/fraud_model.pt are 
    already wired; the work is confined to this file (two device corrections plus a per-epoch checkpointing TODO in the training loop).

3) Run the trainer once against the scaffold as-is—python src/models/train_pytorch.py—to see it fail on the first tensor operation.

4) The end state must include:<br>
    * The script completes successfully and writes a PyTorch state-dict to /root/code/fraud-detection/models/fraud_model.pt.
    * One run exists in the pytorch-training experiment on MLflow, carrying params.device = "cpu" and metrics.final_loss.
    * No bare .cuda() calls remain anywhere in train_pytorch.py.
    * At least two resumable checkpoints exist under /root/code/fraud-detection/checkpoints/ (named ckpt_epoch_*.pt), each a dict carrying the model and optimiser state (so training can resume).
=> first run this command it gives issue 
```test
  python src/models/train_pytorch.py
``` 
Traceback (most recent call last):
  File "/root/code/fraud-detection/src/models/train_pytorch.py", line 101, in <module>
    main()
  File "/root/code/fraud-detection/src/models/train_pytorch.py", line 63, in main
    model = model.cuda()
            ^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/dist-packages/torch/nn/modules/module.py", line 1096, in cuda
    return self._apply(lambda t: t.cuda(device))
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/dist-packages/torch/nn/modules/module.py", line 933, in _apply
    module._apply(fn)
  File "/usr/local/lib/python3.12/dist-packages/torch/nn/modules/module.py", line 964, in _apply
    param_applied = fn(param)
                    ^^^^^^^^^
  File "/usr/local/lib/python3.12/dist-packages/torch/nn/modules/module.py", line 1096, in <lambda>
    return self._apply(lambda t: t.cuda(device))
                                 ^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/dist-packages/torch/cuda/__init__.py", line 484, in _lazy_init
    raise AssertionError("Torch not compiled with CUDA enabled")
AssertionError: Torch not compiled with CUDA enabled


corrected train_pyttorch.py<br>
```bash
  """Feedforward fraud-detection trainer.

Trains a tiny two-layer network on the synthetic transactions CSV,
logs the run to MLflow with `params.device` + `metrics.final_loss`,
and saves the trained weights to `models/fraud_model.pt`.

Data loading, model definition, optimizer setup, loss function, and
the MLflow experiment are already wired. Two things need work:

  1. The current wiring assumes a CUDA GPU is always present. Make
     the device handling runtime-aware so the script runs on whichever
     accelerator the host exposes, and log the device it actually used.
  2. Long training runs need to be resumable — add per-epoch
     checkpointing (TODO inside the loop) so progress survives an
     interruption.
"""
import os

import mlflow
import numpy as np
import pandas as pd
import torch
import torch.nn as nn

TRACKING_URI = "http://localhost:5000"
EXPERIMENT = "pytorch-training"
TRAIN_CSV = "/root/code/fraud-detection/data/train.csv"
MODEL_PATH = "/root/code/fraud-detection/models/fraud_model.pt"
CHECKPOINT_DIR = "/root/code/fraud-detection/checkpoints"

FEATURES = ["amount", "hour", "num_tx_past_day"]
TARGET = "is_fraud"
EPOCHS = 30
LR = 0.01
SEED = 42


class FraudNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(len(FEATURES), 8)
        self.fc2 = nn.Linear(8, 2)

    def forward(self, x):
        return self.fc2(torch.relu(self.fc1(x)))


def main():
    torch.manual_seed(SEED)
    np.random.seed(SEED)

    mlflow.set_tracking_uri(TRACKING_URI)
    mlflow.set_experiment(EXPERIMENT)

    df = pd.read_csv(TRAIN_CSV)
    X = df[FEATURES].values.astype(np.float32)
    y = df[TARGET].values.astype(np.int64)

    X_t = torch.from_numpy(X)
    y_t = torch.from_numpy(y)
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

    model = FraudNet().to(device)

    optimizer = torch.optim.SGD(model.parameters(), lr=LR)
    loss_fn = nn.CrossEntropyLoss()

    os.makedirs(CHECKPOINT_DIR, exist_ok=True)

    with mlflow.start_run(run_name="fraud-mlp"):
        mlflow.log_param("device", device.type)

        xb = X_t.to(device)
        yb = y_t.to(device)

        final_loss = None
        for epoch in range(EPOCHS):
            logits = model(xb)
            loss = loss_fn(logits, yb)
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
            final_loss = float(loss.item())
            print(f"epoch {epoch:02d}  loss={final_loss:.4f}")

            # TODO: every 10th epoch (0, 10, 20), write a resumable
            # checkpoint to CHECKPOINT_DIR/ckpt_epoch_{epoch}.pt. Use
            # torch.save({...}, path) with a dict carrying "epoch", the
            # model "model_state_dict", the optimizer
            # "optimizer_state_dict", and the current "loss" — everything
            # needed to resume training from that point.
            if epoch % 10 == 0:
                checkpoint_path = os.path.join(
                CHECKPOINT_DIR,
                f"ckpt_epoch_{epoch}.pt"
            )

                torch.save(
                {
                    "epoch": epoch,
                    "model_state_dict": model.state_dict(),
                    "optimizer_state_dict": optimizer.state_dict(),
                    "loss": final_loss,
                },
                checkpoint_path,
            )
        mlflow.log_metric("final_loss", final_loss)

        os.makedirs(os.path.dirname(MODEL_PATH), exist_ok=True)
        torch.save(model.state_dict(), MODEL_PATH)
        print(f"Model saved to {MODEL_PATH}")


if __name__ == "__main__":
    main()
```
After resolve issue run again this time it gives 
  1) Model saved to /root/code/fraud-detection/models/fraud_model.pt
  2) The MLflow run will contain:
params.device = cpu
metrics.final_loss = ...
  3) /root/code/fraud-detection/checkpoints/
