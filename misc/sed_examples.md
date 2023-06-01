# sed 示例

## 替换

```bash
cat test.txt 
# 1/1#1
# 22/22/22
# 3/3#3

sed -i 's/22/2/g' test.txt

cat test.txt 
# 1/1#1
# 2/2/2
# 3/3#3
```

## 追加

### 前置追加

```bash
cat test.txt
# mykey=one
# replace=me
# lastvalue=three


sed '/^anothervalue=.*/i before=me' test.txt

cat test.txt
# mykey=one
# before=me
# anothervalue=two
# lastvalue=three
```

### 后置追加

```bash
cat test.txt
# mykey=one
# replace=me
# lastvalue=three

sed '/^anothervalue=.*/a after=me' test.txt

cat test.txt
# mykey=one
# anothervalue=two
# after=me
# lastvalue=three

```

## 删除

### 删除连续行

```bash
cat test.txt 
# 111
# 222
# 333

sed -i '2,3d' test.txt

cat test.txt 
# 111
```

### 删除匹配行

```bash
cat test.txt 
# 111
# 222
# 22
# 333

sed -i "/2/d" test.txt

cat test.txt 
# 111
# 333
```

## 其他

### 替换特殊字符

```bash
cat test.txt 
# 1/1#1
# 2/2$2
# 3/3#3

# 替换斜杠（/）
sed -i 's#2/#2$#g' test.txt 

cat test.txt 
# 1/1#1
# 2$2$2
# 3/3#3

# 替换 $
sed -i 's#2\$#2/#g' test.txt 

cat test.txt 
# 1/1#1
# 2/2/2
# 3/3#3
```


### 结合 find

```bash
cat test/text.txt 
# replace from
# replace to

find ./test -type f  -exec sed -i 's/replace from/replace to/g' {} \;

cat test/text.txt 
# replace to
# replace to
```

### ^M (Windows) 字符转换

```bash
sed -i 's/\r/\n/g' 
```

### 替换多文件

```bash
cat test.txt 
# 1/1#1
# 2/2/2
# 3/3#3
cat test2.txt 
# 1/1#1
# 2/2/2
# 3/3#3

sed -i 's/2/22/g' test.txt test2.txt 

cat test.txt 
# 1/1#1
# 22/22/22
# 3/3#3
cat test2.txt 
# 1/1#1
# 22/22/22
# 3/3#3
```

## 参考

- https://fabianlee.org/2018/10/28/linux-using-sed-to-insert-lines-before-or-after-a-match/
- https://www.folkstalk.com/2013/03/sed-remove-lines-file-unix-examples.html
- https://stackoverflow.com/questions/811193/convert-m-windows-line-breaks-to-normal-line-breaks
