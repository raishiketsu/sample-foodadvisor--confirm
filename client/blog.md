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
helm ReleaseをACKのクラスターにデプロイしたら、


### Stable diffunsionの実行

### 最後に
