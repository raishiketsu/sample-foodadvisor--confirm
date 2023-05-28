### はじめに 
AlibabaCloudにはGPUで機械学習のタスクを実行するときに、効率よくリソースを配分するcGPUという仕組みがあります。
今回はいま話題の生成系AIのStable dffusionをcGPU環境に実行するのを実施して見ようと思います。

### cGPUのご紹介

### Stable diffunsionをcGPU環境にインストール
Stable dffusionはオープンソースのため、GPUなどのコンピューティングリソースがあれば実行できます。一方でオープンソースのため、誰かが環境を用意してくれることがなく、自分でセットアップする必要があります。
本来Stable dffusion環境をセットアップするのが機械学習のエンジニアじゃないと難しいです。有識者がstable-diffusion-webuiという簡単に環境作成できるツールを開発してくれました。

https://github.com/AUTOMATIC1111/stable-diffusion-webui

調べてみたら、k8s環境にstable-diffusion-webuiを簡単にデプロイするためのhelmもあります。

https://github.com/amithkk/stable-diffusion-k8s

最初はhelm chartがあれば簡単にデプロイできると思いきや、実際にこちらのhelmを使おうとするとハマるところがあります。
helm ReleaseをACKのクラスターにデプロイしたら、libglib2.0-0というライブラリがないエラーが出ています。
ローカルにhelm chartに書かれているImageをプルーして、コンテナを実行し、中に入ってlibglib2.0-0を手動でインストールして、
再度commitして自分のコンテナレジストリにプッシュします。
それでlibglib2.0-0がないエラーが解消されましたが、今度は何もエラーが出ない状態でPodが無限ループで再起動しています。
ログに何も吐かれていないため、どうしようもないのでhelmでstable-diffusion-webuiをインストールするのを断念しました。
githubを見たら最新のcommitは半年前のため、頻繁にメンテナンスしていないようです。

方向性を変えて、自分でstable-diffusion-webuiをコンテナ化してデプロイするようにしました。
Dockerfileはこちらです。
```
FROM nvidia/cuda:11.3.0-cudnn8-runtime-ubuntu20.04
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
    libgl1 libglib2.0-0 git wget python3 python3-venv && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

ADD . /stable-diffusion-webui
WORKDIR /stable-diffusion-webui/
RUN ./webui.sh -f can_run_as_root --exit --skip-torch-cuda-test

ENV VIRTUAL_ENV=/stable-diffusion-webui/venv
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

VOLUME /stable-diffusion-webui/models
VOLUME /root/.cache

CMD ["python3", "launch.py", "--listen"]

```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: stable-diffusion
  name: stable-diffusion
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: stable-diffusion
  template:
    metadata:
      labels:
        app: stable-diffusion
    spec:
      containers:
      - args:
        - --listen
        - --skip-torch-cuda-test
        - --no-half
        command:
        - python3
        - launch.py
        image: rais39/stable-diffusion:v2
        imagePullPolicy: IfNotPresent
        name: stable-diffusion
        resources:
          requests:
            cpu: "2"
            memory: 2Gi
          limits:
            aliyun.com/gpu-core.percentage: 25
            aliyun.com/gpu-mem: 5.5
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/alibaba-cloud-loadbalancer-address-type: internet
    service.beta.kubernetes.io/alibaba-cloud-loadbalancer-instance-charge-type: PayByCLCU
  name: stable-diffusion
  namespace: default
spec:
  externalTrafficPolicy: Local
  ports:
  - port: 80
    protocol: TCP
    targetPort: 7860
  selector:
    app: stable-diffusion
  type: LoadBalancer
```

### Stable diffunsionの実行

### 最後に
