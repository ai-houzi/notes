## Mac 杂篇

macOS下安装多个JDK并切换

## 安装
使用`brew cask`安装JDK
```
brew cask info java //首先查看版本信息 
brew cask install java6  #JDK6
brew cask install java8  #JDK8
brew cask install java   #JDK10
```

## 切换脚本
vi .zshrc
```
# Switch JDK
export JAVA_6_HOME="/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home"
export JAVA_8_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_172.jdk/Contents/Home"
export JAVA_10_HOME="/Library/Java/JavaVirtualMachines/jdk-10.0.1.jdk/Contents/Home"
export JAVA_HOME=$JAVA_8_HOME  # Default JDK8
alias jdk6="export JAVA_HOME=$JAVA_6_HOME;java -version"  # to JDK6 
alias jdk8="export JAVA_HOME=$JAVA_8_HOME;java -version"  # to JDK8
alias jdk10="export JAVA_HOME=$JAVA_8_HOME;java -version" # to JDK10
```