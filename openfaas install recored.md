--- openfaas安装v0.18.17（目前serverless里面活跃度最高的）基于k8s至少1.13以上版本
1.下载核心组件
git clone https://github.com/openfaas/faas-netes

2.创建命名空间
kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml

3.创建登陆账号密码(pod端口映射可以参考官网)
kubectl -n openfaas create secret generic basic-auth \
--from-literal=basic-auth-user=admin \
--from-literal=basic-auth-password="$PASSWORD"

4.部署
cd faas-netes && \
kubectl apply -f ./yaml

5.下载cli，并登陆
export OPENFAAS_URL=http://127.0.0.1:31112
echo -n $PASSWORD | faas-cli login --password-stdin

6.最佳实践
通过docker hub完成函数发布
