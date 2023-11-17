# 一、Jenkins服务器和linux服务器互相设置免密登录
# 二、在linux服务器上写sh脚本文件
* 1、新建文件: /usr/local/src/web/restart.sh
````
jarName=$1

PIDS=`ps -ef | grep $jarName | grep -v grep | awk '{print $2}'`

if [ "$PIDS" != "" ]; then
	kill -9 $PIDS
	echo "kill pid = $PIDS"
else
	echo "no process run"
fi

nohup java -server -Xms128m -Xmx256m -XX:NewSize=128m -XX:MaxNewSize=240m  -jar ${jarName} > console.log 2>&1 &
echo 'deploy sucess'
ps -ef | grep ${jarName} | grep -v "grep" 
````
* 2、检查文件格式是否为unix
````
vim restart.sh

如果不是修改为unix
:set ff=unix
:wq
````
* 3、赋权限
````
chmod u+x ./restart.sh
````
# 三、编写Jenkinsfile文件
````
//linux服务器ip
def String serverIp = '10.110.1.112'
//登录的用户名
def String username = 'root'
//打包后jar包所在地址
def String sourcePath = '/var/lib/jenkins/workspace/PMPlatform/TPPlatform/tpplatform-admin/target'
//要上传到linux服务器的地址
def String targetPath = '/usr/local/src/web/'
//jar包名
def String jarName = 'tpplatform-admin.jar'

pipeline{
    agent any
    stages{
        stage('pull code'){
            steps{
                git branch: 'dev', credentialsId: 'e8683dca-a182-4756-849e-c830b839f5c8', url: 'http://10.110.8.205:8032/PMPlatform/TPPlatform.git'
                echo '拉取成功'
            }
        }

        stage('project package'){
            steps{

                sh "echo 开始打包。。。"
                sh "mvn clean package -U -Dmaven.test.skip=true"
                sh "echo 打包成功。"

            }
        }

        stage('deploy') {
            steps {
                script {
                        // 删除linux服务器原有的jar包
                        sh "ssh ${username}@${serverIp} rm -rf ${targetPath}/${jarName};exit"
                        echo '旧文件删除成功'
                        // 上传新的jar包
                        sh """
                        scp ${sourcePath}/${jarName} ${username}@${serverIp}:${targetPath}
                        """
                        echo '文件上传成功'
                        // 运行linux服务器上的sh脚本, 传入jar包名作为变量
                        // 使用ssh命令在远程服务器上运行多行命令时, 需要使用 << EOF将多条命令合并
                        sh """
                            ssh ${username}@${serverIp} << EOF
                            cd ${targetPath}
                            chmod u+x ./restart.sh
                            ./restart.sh ${jarName}
                        """
                }
            }
        }
    }
}
````
# 四、配置Jenkins
* 1、参数化构建过程
````
这里可以动态指定要构建的分支, 本地为单环境部署, 没有配置这项
````
* 2、构建触发器
````
选择 Build when a change is pushed to GitLab. GitLab webhook URL: http://10.110.3.100:8080/project/PMPlatform/TPPlatform
选中
1、Push Events
2、Opened Merge Request Events
3、Comments 同时将Comment (regex) for triggering a build中内容清空
````
* 3、流水线
````
1、选择Pipeline script from SCM
2、在SCM中选择Git
3、在Repositories(仓库)输入gitlab地址和账号密码
4、在Branches to build中指定分支
5、脚本路径指明Jenkinsfile文件位置
````