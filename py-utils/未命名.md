# PyTorch / ONNX Runtime 加速后端选择工具




```python

import torch
//https://github.com/szsy-maker/examination-demo/blob/main/utils.py
//感谢梁学长
def get_device() -> torch.device:
    if torch.cuda.is_available(): # CUDA 加速
        device = torch.device("cuda")
    elif torch.backends.mps.is_available(): # macOS 加速
        device = torch.device("mps")
    else:
        device = torch.device("cpu") # 回退到 CPU
    return device

def getOrtProvider() -> str:
    if torch.backends.mps.is_available(): # 如果macOS MPS能用 那大概 CoreML也能用
        return 'CoreMLExecutionProvider'
    elif torch.cuda.is_available(): # 如果CUDA能用 那大概 TensorRT 也能用
        # return 'TensorRTExecutionProvider'
        return 'CUDAExecutionProvider'
    else:
        return 'CPUExecutionProvider'
```
example
```python

from utils import get_device, getOrtProvider
import onnxruntime as ort

# PyTorch
device = get_device()
# model = MyModel().to(device)
# x = x.to(device)

# ONNX Runtime
provider = getOrtProvider()
sess = ort.InferenceSession(
    "model.onnx",
    providers=[provider, "CPUExecutionProvider"]

)
```