---
title: deploy_on_step.sh
---

##
```bash
#!/bin/bash
set -e # 失败直接结束

uploaded_jar=$1
current_release=$2
deploy_server=$3
deploy_path=$4

echo uploaded_jar is "$uploaded_jar"
echo current release is "$current_release"
echo deploy_server is "$deploy_server"
echo deploy_path is "$deploy_path"

echo --- start upload jar "$uploaded_jar" to "$deploy_server" "$uploaded_jar"  ---
scp "$uploaded_jar" root@"$deploy_server":"$uploaded_jar"
echo --- finished upload jar ---

echo --- start deploy "$deploy_server" ---
ssh root@"$deploy_server" "
  echo 创建release文件夹
  releases_dir=$deploy_path/releases
  shared_dir=$deploy_path/shared
  cd \$releases_dir
  mkdir $current_release
  echo 拷贝jar包到release中
  cd $current_release
  cp $uploaded_jar ./
  echo 拷贝配置文件到release中
  cp -r \$shared_dir/config/. ./
  ln -s \$shared_dir/logs logs
  ln -s \$shared_dir/pids pids
  ln -s \$shared_dir/data data
  ln -s \$shared_dir/script/start.sh start.sh
  ln -s \$shared_dir/script/stop.sh stop.sh
  cd $deploy_path
  ln -sfn releases/$current_release current
  echo 重启服务
  cd $deploy_path/current
  bash -l ./stop.sh
  sleep 5s
  bash -l ./start.sh
  echo delete upload jar
  rm $uploaded_jar
"
echo --- finished deploy "$deploy_server" ---
```
