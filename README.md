## 音乐推荐系统

### data目录

1. 用户画像数据（user_profile.data）
   - 特征：userid，性别，年龄，收入，地域
2. 物品（音乐）元数据（music_meta）
   - 特征：itemid，name， desc，时长， 地域， 标签
3. 用户行为数据（user_watch_pref.sml）
   - userid， itemid，用户对该物品的收听时长，时间（点击，小时）



### pre_base_data

- gen_base.py：把3分数据，合并到一份数据中心，将进行特征处理



### pre_data_for_cb（召回）CB算法

- 目标根据合并数据得出 token，itemid，score 的形式，整理出训练数据，得到item-item的相似矩阵，使用jieba分词提取每首歌曲的关键词和权重，得到结果（token，itemid，score的形式），然后利用协同思想（mr_cf目录），得到ii矩阵。
- 利用 gen_reclist.py 得到粗排的前100条结果，后续插入redis中



### pre_data_for_cf CF算法

- 目标根据合并数据得出 token，itemid，score 的形式，整理出训练数据，得到item-item的相似矩阵，使用jieba分词提取每首歌曲的关键词和权重，得到结果（token，itemid，score的形式），然后利用协同思想（mr_cf目录），得到ii矩阵。
- 利用 gen_reclist.py 得到粗排的前100条结果，后续插入redis中

### mr_cf

- 根据CB和CF的处理数据，借用MR计算II矩阵



### pre_data_for_rankmodel

- 根据基类文件，得出 libsvm的数据格式，用于模型的训练
- 得到用户 和 物品的 特征值属性词典



### rankmodel

喂数据，进行训练，得到模型，用于预测



### rec_server

搭建webpy服务器，获取用户请求的歌曲名字，进行分词，之后查倒排索引表，生出推荐歌曲列表。

组装推荐系统实践：
（1）解析请求：userid，request_itemid
			http://master:9988?userid=01463ded1475e488dfb4e6bc978780cb&itemid=741600204

（2）加载模型：加载model.w model.b
（3）检索候选集合：分别利用cb和cf去redis里面检索数据库，得到item-> item item item推荐候选（300=200+100）
（4）获得用户特征：userid u_fea_dict

（5）获得物品特征：itemid i_fea_dict

（6）打分（sigmoid），排序 prediction = 1/(1+exp(-wx)）

（7）top-n截断
（8）数据包装，itemid->name，返回10条记录
（9）浏览器验证
	http://master:9988/?userid=01463ded1475e488dfb4e6bc978780cb&itemid=741600204