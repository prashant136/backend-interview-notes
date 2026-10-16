

**Question 1: Mac me environment path set karna ?**
```
kisi software ka `.zshrc` ya `bash` me path set ke steps :-

1.) .zshrc open karo
nano ~/.zshrc

2.) Ye line paste karo
export PATH="/opt/homebrew/opt/mysql@8.4/bin:$PATH"

3.) Save karo Nano mein: Ctrl + O ---> Enter

4.) Exit karo ---> Ctrl + X

5.) Refresh / reload karo  --->  source ~/.zshrc

6.) Verify karo ---> which mysql

Expected: /opt/homebrew/opt/mysql@8.4/bin/mysql

Then: mysql --version
```

-----------

**Question 2: Git commands ?**

branch ko kaise merge krte hai ?
rebase and merge ? (kab rebase krte hai) ?
git reset ?
git cherry pick ?
.....