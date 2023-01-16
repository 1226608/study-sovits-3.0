对于https://github.com/innnky/so-vits-svc/tree/32k所实践
非N卡请另寻别处,此总结为win10运行python3.10亲测所写,
我使用了180个8秒的wav片段进行训练,实际上这太少了,
因为我目前只是为了至少能成功就行,完美是以后的事
原作者用wget命令下载文件,说明是linux,
安装ffmpeg并配置exe文件的环境变量
将一段纯人声干声放进一个文件夹内,在文件夹路径沿内衬盖式输入
ffmpeg -i a.wav -f segment -segment_time 8 o_%04d.wav
 文件名                      每段的秒数    输出的文件名
将切出来的片段放入指定文件夹
从第一行cmd命令开始,一路上提示缺什么就安什么
pip install xxxx
以下位置的代码要改,你的python
.py文件中的from开头的那些代码在哪就把from开头的代码添加在哪
Python\Python310\lib\site-packages\parselmouth\adapters\dfp\client.py
改Exception,e:为Exception as e:

Python\Python310\Lib\urllib\__init__.py
添加
from urllib.parse import quote

Python\Python310\Lib\collections\abc.py和__init__.py
都添加
from collections.abc import Mapping
from collections.abc import MutableMapping

提示,不允许用CPU计算
assert torch.cuda.is_available(), "CPU training is not allowed."
安装CUDA和cuDNN,驱动带的CUDA版本可以大于等于CUDA

卸载之前无脑安装的一个CPU版的
pip uninstall torch

安装GPU版的
pip3 install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu117

报错
Distributed package doesn't have NCCL built in
win系统没有NCCL改为gloo
C:\Users\12266\AppData\Local\Programs\Python\Python310\Lib\site-packages\torch\distributed\distributed_c10d.py
在prefix_store = PrefixStore(group_name, store)
的下一排加上backend = "gloo"

训练
显存只吃了1G,跑了一千步.pth文件的大小和修改时间都没变,说明根本就把wav片段跑进去,
检查train.txt内是否有所有模型片段的路径,没有就重新生成预训练出来.pt文件和.npy文件,
train.txt内的有了所有声音片段的路径了

提示爆显存了,说明成功把我的声音片段跑起来了,
,将config.json中的batch_size都改为5,train.py中的num_workers改为2
成功吃了7G的显存,每步所花的时间变慢许多,显卡核心使用率是0,就和在挖矿一样,只吃显存
每提示一次 Saving model and optimizer state at iteration 步数 to ./logs\32k\G_?000.pth 
就代表自动保存了进度.每隔50步数会生成一个新的pth例如G_1000.pth,G_2000.pth,G_3000.pth以此类推
最新进度之前的旧.pth可以删掉,只保留最新的,
每个文件10秒,我觉得跑到和wav文件个数的两倍的步数就够了
推理的时候,用自己训练出的最新的G_?000.pth文件

报错module 'parselmouth' has no attribute 'Sound'
安装parselmouth
报错
pandas 1.5.2 requires pytz>=2020.1, but you have pytz 2015.7 which is incompatible.
输入
pip list
查看安装的版本确实为2015
pip3 uninstall pytz将其卸载
安装
pip install pandas
报错
parselmouth 1.1.1 requires pytz==2015.7, but you have pytz 2022.7.1 which is incompatible.
卸载
pip uninstall pytz
安装
pip install pytz==2015.7
报错
pandas 1.5.2 requires pytz>=2020.1, but you have pytz 2015.7 which is incompatible
卸载
pip uninstall pandas
安装
pip install pytz==2015.7
安装
pip install parselmouth
运行报错
No module named 'pandas'
安装
pip install pandas
报错
parselmouth 1.1.1 requires pytz==2015.7, but you have pytz 2022.7.1 which is incompatible.
卸载
pip uninstall parselmouth
安装
pip install praat-parselmouth
pip install PyAudio
好了可以运行了
不要用音色差别大的作为模仿对象,最好是年龄和性别都相近,以至于最好是让trans = [0] 
真想填的话在先在Melodyne 5里搜索,把模仿对象的声音调到这么高或这么低试试会怎么样
+或-4是最多了,在没别的办法的情况下得到纯净说话的干声推荐微软Azure,歌声推荐VOCALOID5及以上
用歌声训模仿歌声,说话训模仿说话是最好的
