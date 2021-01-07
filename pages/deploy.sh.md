---
title: deploy.sh
---

##
```bash
#!/bin/bash
# 部署目录
set -e # 失败直接结束

# 工作目录
WORK_DIR="$(dirname "$(realpath -s "$0")")"
# 项目目录
PROJECT_PATH=$(dirname "$WORK_DIR")
# 项目名称
PROJECT_NAME=$(basename "$PROJECT_PATH")
# 父项目路径
PROJECT_PARENT_PATH=$(dirname "$PROJECT_PATH")
# jar包上传路径
UPLOADS_PATH="/root/uploads"
# 服务器部署目录：统一在/eemp/servers下以项目名称分隔
DEPLOY_PATH="/eemp/servers/$PROJECT_NAME"
# 跳板机地址
STEP_SERVER_ADDR=$(basename "$(ls -t "$WORK_DIR"/*.step.host | head -1)" .step.host)
# 当前版本号
CURRENT_RELEASE=$(date +%Y%m%d%H%M%S)

# jar包构建
echo --- start build project eemp-core ---
cd "$PROJECT_PARENT_PATH/eemp-core"
mvn clean install -Dmaven.test.skip=true
echo --- finish build project eemp-core ---

echo --- start build project "$PROJECT_NAME" ---
cd "$PROJECT_PATH"
mvn clean package -Dmaven.test.skip=true
echo --- finish build project "$PROJECT_NAME" ---

# jar包上传
echo --- start copy jar to step server "$STEP_SERVER_ADDR" ---
jar_file_full_path=$(ls -t "$PROJECT_PATH"/target/*.jar | head -1)
jar_file_name=$(basename "$jar_file_full_path")
upload_jar_path="$UPLOADS_PATH/$jar_file_name"
scp "$jar_file_full_path" root@"$STEP_SERVER_ADDR":"$upload_jar_path"
echo --- finished copy jar to step server "$STEP_SERVER_ADDR" ---

cd "$PROJECT_PATH/deploy"
for deploy_server_file in deploy_servers/*.server.host
do
  deploy_server=$(basename "$deploy_server_file" .server.host)
  echo --- start deploy server "$deploy_server" on "$DEPLOY_PATH" with release "$CURRENT_RELEASE" ---
  ssh root@"$STEP_SERVER_ADDR" "bash -s $upload_jar_path $CURRENT_RELEASE $deploy_server $DEPLOY_PATH" < deploy_on_step.sh
  echo --- finished deloy server "$deploy_server" on "$DEPLOY_PATH" with release "$CURRENT_RELEASE" ---
done

echo clean upload jar $upload_jar_path on step server $STEP_SERVER_ADDR
ssh root@"$STEP_SERVER_ADDR" "rm $upload_jar_path"
```
