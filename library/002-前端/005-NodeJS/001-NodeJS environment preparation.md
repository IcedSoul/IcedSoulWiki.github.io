# NodeJS environment preparation

## NVM

It is not recommended to install `NodeJs` directly, after install `nvm` , you can use it to install and mange any verion of `NodeJS` .

### Linux/Mac OS [1]

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash
```

Then you can run `nvm --version`to check the version of  `nvm` you just installed.

### Windows [2]

Just download the installer and run it!



If you want to install `NodeJS 8`,just execute:

```shell
nvm install 8.16.2
nvm use 8.16.2
```



## Yarn

### ubuntu

```shell
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt update && sudo apt install --no-install-recommends yarn
```





## Reference

1.  https://github.com/nvm-sh/nvm 
2.  https://github.com/coreybutler/nvm-windows 
3.  https://yarnpkg.com/en/docs/install#alternatives-stable 
4.  https://www.ostechnix.com/install-node-js-linux/