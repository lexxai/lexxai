---
layout: post
title: "Нотатка: pyzbar, pyinstaller, poetry, python - missing lib"
date: 2023-09-08 08:47:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/09/pyzbar-pyinstaller-poetry-python.html
---

При використанні бібліотеки pyzbar, її компоненти не додаються до exe файлу створеного за допомогою pyinstaller.  
Ось розв'язання проблеми. У мене середовище: Python 3.11.5 та poetry.  

```
pyinstaller "../some_app/main.py" --clean --name some_app --onefile --version-file "../versionfile.txt" --add-binary "%VIRTUAL_ENV%\Lib\site-packages\pyzbar\*.dll;pyzbar"
```

EXE файл після цього самодостатнім є, додаткових dll додавати не треба.

|  |
| --- |
|  |
| Перевірено у [Windows Sandbox](/2019/09/windows-10-version-1903-sanbox.html). |

За матеріалами.

* [python - Got this error on my pyzbar program after I converted to exe with pyinstaller - Stack Overflow](https://stackoverflow.com/questions/67886525/got-this-error-on-my-pyzbar-program-after-i-converted-to-exe-with-pyinstaller)

P.S. Build version.txt - build-version.py  

```
import pyinstaller_versionfile
from configparser import ConfigParser
import sys
from pathlib import Path


parser = ConfigParser()
# read config file
config_setup=Path("..\setup.cfg")
if config_setup.is_file():
    parser.read(config_setup)
    ver = parser["metadata"].get("version", "0.0.1")
else:
    config_setup=Path("..\pyproject.toml")
    if config_setup.is_file():
        parser.read(config_setup)
        ver = parser["tool.poetry"].get("version", "0.0.1")
        ver = ver.strip('"')

if len(sys.argv) > 1:
    filename = Path(sys.argv[1])
    if filename.is_file():
        new_fn = filename.with_stem(f"{filename.stem}_{ver}")
        filename.rename(new_fn)
else:
    pyinstaller_versionfile.create_versionfile(
        output_file="..\\versionfile.txt",
        version=f"{ver}.0",
        company_name="lexxai",
        file_description="Some Python App of lexxai",
        internal_name="some_app",
        legal_copyright="https://github.com/lexxai",
        original_filename="some_app.exe",
        product_name="some_app",
    )
    print(f"Done: versionfile.txt in parent folder. version='{ver}.0'")
```

Build script: script/build.cmd

```
python build-version.py

mkdir "../pyinstall"
ERASE "../pyinstall" /S/Q
PUSHD "../pyinstall"

pyinstaller "../some_app/main.py" --clean --name some_app --onefile --version-file "../versionfile.txt" --add-binary "%VIRTUAL_ENV%\Lib\site-packages\pyzbar\*.dll;pyzbar"
POPD
python build-version.py ../pyinstall/dist/some_app.exe
```
