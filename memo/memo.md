# リンク

- プロジェクトページ 
https://metagen.ai/transflower.html
- arxiv 
https://arxiv.org/abs/2106.13871
- transflower-lightning
https://github.com/guillefix/transflower-lightning
- 現在の継続的な開発ページ
https://github.com/MetaGenAI/multimodal-transflower


# 備忘

## とりあえずColaboであそんでみる

https://colab.research.google.com/drive/1SBEJZp3TdVbgjAP9pwsTPqaefK3QuUVj#scrollTo=P6_PREvgZtXA

> Parov_Stelar_-_The_Phantom_Official_Audio-mE8YOyxJ
training/experiments/transflower_expmap_old/version_15/checkpoints/epoch=311-step=630988.ckpt
Traceback (most recent call last):
  File "inference/generate.py", line 76, in <module>
    exp_opt = yaml.load(open(str(checkpoint_dir)+"/hparams.yaml","r").read())
TypeError: load() missing 1 required positional argument: 'Loader'

https://mojel.blog.fc2.com/blog-entry-7.html
これっぽい. => 動いた/

## とりあえずローカルで遊んでみる
(On windows projects/motion/transflower-lighting)

- Colaboで遊んだ時のダウンロード物をローカルに一括ダウンロード

### 環境構築
> python -m venv .venv  

- 先にインストールしないとエラーがでた.
> pip install Cython numpy
- 
> pip install -r requirements.txt

- pip install madmom
>       error: Microsoft Visual C++ 14.0 or greater is required. Get it with "Microsoft C++ Build Tools": https://visualstudio.microsoft.com/visual-cpp-build-tools/
      [end of output]

- vscodeをinstallする
https://github.com/CPJKU/madmom/issues/478

- pip install madmom
正常に終了. 

- torchがcuda使えないので
pip uninstall torch

- もう一回インストール
> pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu116

- 確認
> import torch  
> torch.cuda.is_available()  
True

