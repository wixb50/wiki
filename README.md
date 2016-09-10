# Elegance Tse Wiki.

Stay hungry,Stay foolish.


## 安装(python2)

```
$ pip install simiki
$ pip install fabric
```

## 本地化步骤

```
git clone https://github.com/wixb50/wiki.git
cd wiki && mkdir output && cd output
git clone https://github.com/wixb50/wiki.git .
git checkout gh-pages
cd ..
```

## 提交步骤

```
simiki g
git push origin master

cd output
git push origin gh-pages
```
