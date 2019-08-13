## Git

### 找回丢失的commint 

```
//查找
git fsck --lost-found
//显示提交信息
git show id
//合并到当前分支
git merge id
```

### 撤销commit，但未git push的命令

```
//完成撤销,同时将代码恢复到前一commit_id 对应的版本 
git reset –hard id 
//完成Commit命令的撤销，但是不对代码修改进行撤销，可以直接通过git commit 重新提交对本地代码的修改
git reset id 
```

### submodule

#### 初始化和更新

`git submodule update --init --recursive --remote`



## Cocoapods

### 添加源 

`pod repo add name http://xxx/xxx/pod_specs.git`

### 移除源

 `pod repo remove name`

### 更新源

 `pod repo update`

更新指定源 ` pod repo update /Users/username/.cocoapods/repos/master/Specs`

### 安装库

`pod install --no-repo-update`

### 更新库

`pod update --no-repo-update`



### 私有库



**创建cocopods库模板，带demo**

```
pod lib create name
```

**验证podspec文件命令**

```
$ pod lib lint  SPEC_NAME.podspec  # --allow-warnings 
```

**验证过程中也可能由于代码问题，出现代码编译方面的error，可以添加`--verbose`来查看详细的log：**

```
pod spec lint SPEC_NAME.podspec --verbose
```

**验证podspec文件的正确性后，可以在本地先进行一次安装，没问题后再推送到仓库，修改podfile，指定podspec地址为本地的地址:**

```
pod 'PodTestAlertView', :podspec => '/Users/masaike/.cocoapods/repos/TestRepository/PodTe
```

**本地验证通过后就可以上传版本了**

```
pod repo push MyRepo MyAdditions.podspec
```

### 打包插件cocoapods-binary

详细用法：[来源](https://zhuanlan.zhihu.com/p/36439065)

1. 安装插件 `gem install cocoapods-binary`
2. 在你的 Podfile 做上面加上 `plugin 'cocoapods-binary'` ，并在想要二进制化的 pod 后面加上 `:binary => true` ( 或者直接使用 `all_binary!` 让全部生效 )
3. pod install

## Carthage

### 打包Framework 

`carthage build --no-skip-current`

## Mac

### 查看Framework架构

`lipo -info 二进制路径`

### 模拟器列表

`xcrun simctl list`

### 删除无效模拟器

`xcrun simctl delete unavailable`

