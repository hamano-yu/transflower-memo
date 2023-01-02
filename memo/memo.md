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

On Ubuntu22.04 projects/motion/transflower-memo

- requirementsのインストール

`   ERROR: Failed building wheel for mpi4py`

- openmpiのインストール
```
sudo apt-get -y install openmpi-bin  
sudo apt-get -y install libopenmpi-dev
```
- もう一回、  
` pip install -r requirements.txt`

- youtube-dl
```
sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl  
sudo chmod a+rx /usr/local/bin/youtube-dl  
```
- テストしてみる  

`./feature_extraction/audio_feature_extraction_test.sh songs/  `

> ImportError: cannot import name 'comb_filters' from partially initialized module 'madmom.audio' (most likely due to a circular import) (/home/kitsume/Desktop/projects/motion/transflower-memo/feature_extraction/madmom/audio/__init__.py)  

エラー  
madmomのインストール手順にしたがえと
https://github.com/CPJKU/madmom/issues/259  

` pip install pyaudio `

> ERROR: Could not build wheels for pyaudio, which is required to install pyproject.toml-based projects  

` sudo apt install build-essential portaudio19-dev` 
もう一回、pip install pyaudio   
```
> pip install pyaudio
> pip install pyfftw   
```
もう、一回madmom install   

` pip install madmom ` 

やっぱりだめ。

- ん、feature_extractionにmadmomがおるぞ、なんやこいつ..

> madmom.audio.cepstrogramがpip install 経由ではいない.
```
>>> madmom.audio.c
> madmom.audio.chroma        madmom.audio.comb_filters  
```
- madmomをソースからinstallしてみる.

https://madmom.readthedocs.io/en/latest/installation.html  
なるほど、madmom.audio.cepstrogramは、mfccsブランチにいるのね.
https://github.com/CPJKU/madmom/issues/380


``` 
git checkout mfccs
python setup.py develop --user
```

- cepstrogramがいる
```
>>> madmom.audio.c
madmom.audio.cepstrogram   madmom.audio.chroma        madmom.audio.comb_filters  
>>> madmom.audio.c
```

- ソースコードを変更する. feature_extraction.madmomになっている部分をそのままmadmom importにかえる

```
~~import feature_extraction.madmom as madmom~~
import madmom as madmom
```

`./feature_extraction/audio_feature_extrancion_test.sh songs/`  
動いた. なるほど、songs/フォルダ配下にある、.wavファイルをいい感じに.wavにしてくれるっぽい.

- リソースの準備

``` colabの.ipynbの以下のリソースをダウンロードして、localに配置する.

#@title Prepare transflower and dependencies
#@markdown Run this cell to download transflower to your drive (only will if it hasn't downloaded already), and prepare dependencies
%%capture
%%bash
if [[ -z "$(ls -A .)" ]]; then
  git clone https://github.com/guillefix/transflower-lightning .
  mkdir songs
  mkdir training/experiments
  if [ "$download_Transflower" == "True" ]; then
  gsutil -m cp -r gs://metagen/models/transflower_expmap_old training/experiments/
  fi
  if [ "$download_Transflower_finetune" == "True" ]; then
  gsutil -m cp -r gs://metagen/models/transflower_expmap_finetune2_old training/experiments/
  fi
  if [ "$download_Transflower_2nd_checkpoint" == "True" ]; then
  gsutil -m cp -r gs://metagen/models/transflower_expmap training/experiments/
  fi
  if [ "$download_Transformer" == "True" ]; then
  gsutil -m cp -r gs://metagen/models/transformer_expmap1 training/experiments/
  fi
  if [ "$download_MoGlow" == "True" ]; then
  gsutil -m cp -r gs://metagen/models/moglow_expmap training/experiments/
  fi
  gsutil -m cp -r gs://metagen/scalers/* songs/
  gsutil -m cp -r gs://metagen/seeds/* songs/
fi
pip install -r requirements.txt
pip install mido madgrad
apt-get install ffmpeg
sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
sudo chmod a+rx /usr/local/bin/youtube-dl
```

### 作業

- データダウンロード
colaboノートの以下の部分を実行. 
urlはこれに変更. https://www.youtube.com/watch?v=0rUmLs3hul8

``` 
youtube_url = "https://www.youtube.com/watch?v=0rUmLs3hul8"
if download_from_youtube:
  !export youtube_url
  !youtube-dl -x --audio-format wav --restrict-filenames $youtube_url
  filename=!youtube-dl -x --audio-format mp3 --get-filename --restrict-filenames $youtube_url
  filename = os.path.splitext(filename[0])[0]+".wav"
  filename_new = filename[:50].replace(".","_")+".wav"
  !mv $filename songs/$filename_new
  filename = filename_new
  filename=filename[:-4] # removing final extensions
else:
  res=files.upload()
  filename=list(res.keys())[0]
  os.rename(filename,"songs/"+filename)
  filename=filename[:-4] # removing final extensions

print(filename)
```

- feature_extraction

`./feature_extraction/audio_feature_extraction_test.sh songs/`


- generate dance
colaboノートの以下実行.

```
#@title Generate dance 
#@markdown Generate dance ^^ 💃

#@markdown If it shows some nans ignore them <small>(its some issue if motion seed clip is shorter than the song)</small>

#@markdown You can choose one of the following motion seeds, that will affect the style (you can also input a custom one, but that's more advanced)
motion_seed = 'Casual' #@param ["Free style", "Casual", "Hip Hop", "Break dance", "Groovenet", "Casual2"]

#@markdown You can choose among these two models. If you want to try the baselines MoGlow or Transformer from the paper, you'd need to have downloaded them in the above step
model = 'Transflower' #@param ["Transflower", "Transflower fine tuned", "Transformer", "MoGlow", "Transflower_second_checkpoint"]
# model = 'Transflower' #@param ["Transflower", "Transflower fine tuned"] {allow-input: true}
generate_bvh = True #@param {type:"boolean"}
generate_video = True #@param {type:"boolean"}
short_test_run = False #@param {type:"boolean"}
seed_options = ["Free style", "Casual", "Hip Hop", "Break dance", "Groovenet", "Casual2"]
seed = str(seed_options.index(motion_seed) + 1)
!chmod +x ./script_generate.sh
!cp 'songs/seed'{seed}'.npy' 'songs/'{filename}'.expmap_scaled_20.npy'

if model == "Transflower":
  model_string = "transflower_expmap_old"
elif model == "Transflower fine tuned":
  model_string = "transflower_expmap_finetune2_old"
elif model == "MoGlow":
  model_string = "moglow_expmap"
elif model == "Transformer":
  model_string = "transformer_expmap1"
elif model == "Transflower_second_checkpoint":
  model_string == "transflower_expmap"
else:
  raise NotImplementedError("model "+model+" not implemented!")

extra_args = ""
if generate_bvh:
  extra_args += "--generate_bvh "
if generate_video:
  extra_args += "--generate_video "
if short_test_run:
  extra_args += "--max_length=512 "

!./script_generate.sh {model_string} {filename} {extra_args}--data_dir=songs;
```
以下エラー、https://mojel.blog.fc2.com/blog-entry-7.html


```
    exp_opt = yaml.load(open(str(checkpoint_dir)+"/hparams.yaml","r").read())
TypeError: load() missing 1 required positional argument: 'Loader'
```

- 3分くらいで完了したっぽい。

- 確認. 以下を実行.
```
#@title Visualize
#@markdown This will copy the generated dance to a server and open a visualizer
# %%capture
import uuid
from IPython.display import Javascript
hash = str(uuid.uuid1())
!echo $(echo {filename})_{hash}
!cp inference/generated/{model_string}/videos/{filename}.bvh inference/generated/{model_string}/videos/{filename}_{hash}.bvh
!gsutil cp inference/generated/{model_string}/videos/{filename}_{hash}.bvh gs://metagendance/
!cp inference/generated/{model_string}/videos/{filename}.mp4.mp3 inference/generated/{model_string}/videos/{filename}_{hash}.bvh.mp3
!gsutil cp inference/generated/{model_string}/videos/{filename}_{hash}.bvh.mp3 gs://metagendance/
url = 'https://guillefix.github.io/bvh_visualizer/?name='+filename+'_'+hash
print("Opening "+url)
display(Javascript('window.open("{url}");'.format(url=url)))
```

- おおうごいたうごいた

- HipHopに変更してみる

## windowsやっぱりめんどいわ  一旦、Ubuntuで
~~ (On windows projects/motion/transflower-lighting) ~~ 

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

### データ準備

- colaboのgsutilのコマンドのパスを見ながら、ダウンロードしたアセットを移動する.

- 