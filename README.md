A simple web app to search for **economics papers** in **economics journals**.

https://econ-paper-search.streamlitapp.com/

### 项目简介（中文）
- **这是一个经济学论文搜索应用**，聚焦主流经济学期刊。
- 支持：关键词检索（标题/摘要/作者可选）、期刊集合筛选、年份范围筛选、按年份排序、一次展示大量结果。

### 数据来源
- **来源网站**：数据来自 **RePEc**（IDEAS），月度更新。
- 链接： [RePEc (IDEAS)](https://ideas.repec.org/)
- 说明：本仓库仅包含前端检索应用，**爬虫/抓取代码未包含**。数据以 CSV 文件形式存放于 `Data/` 目录。

### 数据文件结构与命名约定
- 目录：`Data/`
- 已使用文件（示例）：`papers_b2000.csv`、`papers_2000s.csv`、`papers_2010s.csv`、`papers_2015s.csv`、`papers_2020s.csv`、（代码中也读取了）`papers_2025s.csv`
- 命名建议：按年代或自定义分段生成 CSV，列至少包含：`title, authors, abstract, url, journal, year`。

### 在哪里修改“新增年份/数据段”
应用端读取数据在 `Code/app.py`：

现有读取与合并逻辑位置：
```16:25:/Users/ynbsztl/Documents/Python/github_pub_repo/Econ-Paper-Search/Code/app.py
def load_data_and_combine():
    args = {"dtype": {"year": "Int16"}, "usecols": ["title", "authors", "abstract", "url",  "journal", "year"]}
    df1 = pd.read_csv("Data/papers_b2000.csv", **args)
    df2 = pd.read_csv("Data/papers_2000s.csv", **args)
    df3 = pd.read_csv("Data/papers_2010s.csv", **args)
    df4 = pd.read_csv("Data/papers_2015s.csv", **args)
    df5 = pd.read_csv("Data/papers_2020s.csv", **args)
    df6 = pd.read_csv("Data/papers_2025s.csv", **args)    
    df = pd.concat([df1, df2, df3, df4, df5, df6], axis=0)
    return df
```

更新时间戳（用于缓存失效）的位置：
```46:48:/Users/ynbsztl/Documents/Python/github_pub_repo/Econ-Paper-Search/Code/app.py
def load_data():
    update_timestamp = os.path.getmtime("Data/papers_2020s.csv")
    return load_data_cached(update_timestamp)
```

UI 年份范围配置位置：
```344:351:/Users/ynbsztl/Documents/Python/github_pub_repo/Econ-Paper-Search/Code/app.py
    year_min = 1900
    year_max = 2026
    
    c1, c2, c3, c4 = form.columns(4)
    year_begin = c1.number_input('Year from', value=1980, min_value=year_min, max_value=year_max)
    year_end = c2.number_input('Year to', value=year_max, min_value=year_min, max_value=year_max)
```

#### 增加一个新的年份数据段（示例）
1) 将新 CSV（例如 `Data/papers_2030s.csv`）放入 `Data/`。
2) 在 `load_data_and_combine()` 中追加一行读取，并在 `pd.concat([...])` 中加入：
```python
# 新增一行读取
df7 = pd.read_csv("Data/papers_2030s.csv", **args)

# 在 concat 中追加 df7
df = pd.concat([df1, df2, df3, df4, df5, df6, df7], axis=0)
```
3) 在 `load_data()` 中将用于刷新缓存的时间戳文件，改为你会随数据更新的最新文件：
```python
update_timestamp = os.path.getmtime("Data/papers_2030s.csv")
```
4) 如需放宽 UI 可选年份上限，修改：
```python
year_max = 2035  # 根据你的数据上限设置
```

提示：如果你不想按年代拆分文件，也可以合并成一个大 CSV，只需在 `load_data_and_combine()` 中改为读取一个文件即可。

### 在哪里修改“期刊列表/集合”
期刊集合定义在 `Code/app.py` 的 `main()` 中：

基础列表与扩展列表：
```311:329:/Users/ynbsztl/Documents/Python/github_pub_repo/Econ-Paper-Search/Code/app.py
    js = ['aer', 'jpe', 'qje', 'ecta', 'restud',
          'aejmac', 'aejmic', 'aejapp', 'aejpol', 'aeri', 'jpemic', 'jpemac',
          'restat', 'jeea', 'eer', 'ej',
          'jep', 'jel', 'are',
          'qe', 'jeg',
          'jet', 'te', 'joe',
          'jme', 'red', 'rand', 'jole', 'jhr',
          'jie', 'ier', 'jpube', 'jde',
          'jeh','jue',
          'jhe',
          ]
    js_ = ["jae","geb","jinde","jlawe","jebo","ee","ectt","ehr","eeh","imfer","ecot","jmcb","edcc","sje","ecoa",
            "jaere","jeem","wber","ijio","jleo","le","jpope","qme","ei","jedc","cej","obes","jems","jes","jmate",
            "rsue","eedur","jhc","efp","aler","jbes",
            "jf","jfe","rfs","ms","jbf","smj","rp","bpea","er","ijgt","ntj","md","jdeme","oxe","jei","riw","ajhe"
            ]
    if full_journal:
        js += js_
```

分类集合（可用于在下拉中选择如 `top5`、`general`、`survey`）：
```329:336:/Users/ynbsztl/Documents/Python/github_pub_repo/Econ-Paper-Search/Code/app.py
    js_cats = {"all": js,
               "top5": ['aer', 'jpe', 'qje', 'ecta', 'restud'],
               "general": ['aer', 'jpe', 'qje', 'ecta', 'restud', 'aeri', 'restat', 'jeea', 'eer', 'ej', 'qe'],
               "survey": ['jep', 'jel', 'are', ]
               }
```

下拉可选与展开逻辑：
```335:343:/Users/ynbsztl/Documents/Python/github_pub_repo/Econ-Paper-Search/Code/app.py
    journals = form.multiselect("Journals",
                                js_cats_keys+js, js)
    # 如果选择了分类键（如 top5），则展开为对应的期刊列表
    js_temp = set(journals) & set(js_cats_keys)
    if js_temp:
        for c in js_temp:
            journals += js_cats[c]
        journals = set(journals)
```

#### 新增一个期刊缩写（示例）
1) 在基础列表 `js` 中加入你的缩写（例如新增 `newj`）：
```python
js = [
    # ... 保留原有条目
    'newj',  # 新增
]
```
2) 如果该期刊属于某分类（如 `general`），把 `newj` 同时加入 `js_cats["general"]`：
```python
js_cats = {
    "all": js,
    "top5": [...],
    "general": [
        # ... 保留原有条目
        'newj',  # 新增到分类
    ],
    "survey": [...]
}
```
3) 侧边栏的“Journal Abbreviations”是说明文本（非功能性），如果希望给用户显示新缩写的全称与说明，可在该 Markdown 段落中补充对应行。

侧边栏位置：
```165:261:/Users/ynbsztl/Documents/Python/github_pub_repo/Econ-Paper-Search/Code/app.py
    st.sidebar.header("Journal Abbreviations")
    st.sidebar.markdown("""
    <div style="color: green; font-size: small">
    aejapp: AEJ Applied Economics<br>
    # ... 中间省略，多数缩写在此声明
    </div>
    """, unsafe_allow_html=True)
```

### 为什么不用 Google 搜索？
- 结果里经常混入非经济学论文
- 不能筛选特定期刊集合
- 不能按发表年份排序
- 很难在一个页面看到大量结果

### 注意事项
- 由于 Streamlit 的一些未知原因，个别浏览器可能出现空白页。可尝试清理缓存/更换浏览器。

### 贡献与期刊建议
- 期刊集合带有明显主观性。如果你希望加入新的期刊，欢迎提 issue。

### git command
```bash
cd "/Users/ynbsztl/Documents/Python/github_pub_repo/Econ-Paper-Search"
git add .
git status
git commit -m 'version_1.3_stable'
git push
```

