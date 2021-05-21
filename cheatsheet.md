Register conda to be used with jupyter:
```
#!/bin/bash
# create conda environment for lab
conda create -n ignite_ml_lab python=3.6 anaconda -y
conda activate ignite_ml_lab
python -m ipykernel install --user --name ignite_ml_lab --display-name "ignite_ml_lab"
# install dependencies
pip install …
```

Conda cheatsheet: [pdf](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf)

Plot correlation between categorical features and response variable:
```
def encode(frame, feature):
    ordering = pd.DataFrame()
    ordering["val"] = frame[feature].unique()
    ordering.index = ordering.val
    ordering["spmean"] = frame[[feature, "SalePrice"]].groupby(feature).mean()["SalePrice"]
    ordering = ordering.sort_values("spmean")
    ordering["ordering"] = range(1, ordering.shape[0]+1)
    ordering = ordering["ordering"].to_dict()
    
    for cat, o in ordering.items():
        frame.loc[frame[feature] == cat, feature+'_E'] = o
    
categorical_vars_E = []
for q in categorical_vars:  
    encode(df_housing, q)
    categorical_vars_E.append(q+"_E")

def boxplot(x, y, **kwargs):
    sns.boxplot(x = x, y = y)
    x = plt.xticks(rotation = 90)

data = pd.melt(df_housing, id_vars = ["SalePrice"], value_vars = categorical_vars_E)
g = sns.FacetGrid(data, col = "variable",  col_wrap = 5, sharex = False, sharey = False)
g = g.map(boxplot, "value", "SalePrice")
```
