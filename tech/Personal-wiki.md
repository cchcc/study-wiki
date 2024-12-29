## 개인위키 세팅하기
### gollum + bitbucket
깃헙은 private가 유료라서 private가 무료인 bitbucket도 괜찮음

1. install ruby

2. install gollum
  ```
brew install icu4c
sudo gem install charlock_holmes -- --with-icu-dir=/usr/local/opt/icu4c
sudo gem install gollum
sudo gem install github-markdown
```  

3. run gollum
  ```
mkdir my-wiki
cd my-wiki
git init
touch Home.md
git add -A
git commit -m "Create Home.md"
gollum
```
4. add bitbucket  
  ```
git remote add origin https://사용자명@bitbucket.org/사용자명/my-wiki.git/wiki
```

[http://nolboo.kim/blog/2013/12/17/markdown-wiki-bitbucket-gollum/](http://nolboo.kim/blog/2013/12/17/markdown-wiki-bitbucket-gollum/)  
[https://github.com/gollum/gollum/wiki/Installation](https://github.com/gollum/gollum/wiki/Installation)
