# 阿里云docker容器服务器安装nextcloud步骤

## 安装ossfs
- 阿里云docker服务器基础服务器为centos7
- 进入`root`用户
- 下载  ossfs `wget http://gosspublic.alicdn.com/ossfs/ossfs_1.80.6_centos7.0_x86_64.rpm`
- 安装  ossfs `yum install ossfs_1.80.6_centos7.0_x86_64.rpm`
- 配置账号信息`echo bucktName:keyId:secrectKey > /etc/passwd-ossfs`
- 设置账号信息文件夹权限640`chmod 640 /etc/passwd-ossfs`
- 安装`docker-compose`: 
    > curl -L https://get.daocloud.io/docker/compose/releases/download/v2.4.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
    >  chmod +x /usr/local/bin/docker-compose
    > ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
- `docker-compose up`启动nextcloud
- 启动之后访问网址，安装好nextcloud之后`docker-compose stop`所有服务
- 进入nextcloud app挂载文件处`/home/admin/ossfs/nextcloud/data/`
- 将此处文件先移动到别处`mv ./* /opt` 此处文件夹有隐藏文件，建议`ls -la`查看之后手动移动，一定要保证当前`data`文件夹为空文件夹
- 返回上一级`/home/admin/ossfs/nextcloud`，执行`ls -la`查看文件夹所属的uid与gid
    > drwxrwx---  1   82   82     0 Jan  1  1970 data #这是我的云服务器uid为82 gid为82

- 执行挂载命令`ossfs bucketName /home/admin/ossfs/nextcloud/data -ourl=oss-cn-hongkong-internal.aliyuncs.com -ouid=82 -ogid=82 -oumask=007 -o allow_other`

- 启动nextcloud `docker-compose up -d`

- 完成挂载

## 当前问题
- 云服务器挂载oss之后上传速度被限制在了`1.4m` 还没有找到解决方案


## 问题解决 
- app不能登录问题
- `docker-compose exec -u 82 app /bin/sh` 修改`config`目录下的`config.php`
- 修改为以下内容

```php
<?php
$CONFIG = array (
  'memcache.local' => '\\OC\\Memcache\\APCu',
  'apps_paths' =>
  array (
    0 =>
    array (
      'path' => '/var/www/html/apps',
      'url' => '/apps',
      'writable' => false,
    ),
    1 =>
    array (
      'path' => '/var/www/html/custom_apps',
      'url' => '/custom_apps',
      'writable' => true,
    ),
  ),
  'memcache.distributed' => '\\OC\\Memcache\\Redis',
  'memcache.locking' => '\\OC\\Memcache\\Redis',
  'redis' =>
  array (
    'host' => 'redis',
    'password' => '',
    'port' => 6379,
  ),
  'instanceid' => 'ocq4iwpl1inv',
  'passwordsalt' => 'zWOcgi3ls4jm3nS0qZBB68fdywS+Z/',
  'secret' => '04Wy9htCvw6wQxsHEHpF5Q3m9gfdiSbw2IGK57CLFWiSgtvI',
  'trusted_domains' =>
  array (
    0 => 'pan.kws.knd.com.cn',
    1 => '47.243.249.3:443',
  ),
  'overwriteprotocol' => 'https',
  'overwritehost' => 'pan.kws.knd.com.cn',
  'datadirectory' => '/var/www/html/data',
  'dbtype' => 'pgsql',
  'version' => '23.0.11.1',
  'overwrite.cli.url' => 'https://pan.kws.knd.com.cn',
  'dbname' => 'nextcloud',
  'dbhost' => 'db',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'dbuser' => 'oc_admin',
  'dbpassword' => 'ReDiTQAqXo1KRxxx0SGZO00ZZPyM96',
  'installed' => true,
);
```