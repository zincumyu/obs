## 快速写入python指令
```powershell
@'
import sys

def main():
    print("created by powershell")
    print(sys.version)

if __name__ == "__main__":
    main()
'@ | Set-Content .\demo.py
```

#小结一下这个FastAPI和request