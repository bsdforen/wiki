# Sphinx Wiki für BSDForen.de

![build](https://github.com/bsdforen/wiki/workflows/build/badge.svg)
![deploy](https://github.com/bsdforen/wiki/workflows/deploy/badge.svg)

## Virtuelle Python Umgebung erstellen

~~~bash
~% python -m venv sphinx
~% source sphinx/bin/activate
~% pip install --upgrade pip
~~~

## Git Repo klonen

~~~bash
~% git clone https://github.com/bsdforen/wiki
~% cd wiki
~~~

## Sphinx in die virtuelle Python Umgebung installieren

~~~bash
~% pip install -U -r requirements.txt
~~~

## Dokumentation aus den Quellen erstellen

~~~bash
~% sphinx-build -d build/doctrees -b html source build/html
~% firefox build/html/index.html
~~~

oder

~~~bash
~% sphinx-autobuild -i "*.swp" -i "*.swx" -b html source build/livehtml
~% firefox http://127.0.0.1:8000
~~~
