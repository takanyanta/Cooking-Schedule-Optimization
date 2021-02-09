# Cooking-Optimization
Linear Programming, Optimization, Production Engineering(Industrial Engineering)

## 1. Purpose

* To explore optimal procedure of cooking
* To minimize cumulative cooking time

## 2. Assumption

* Define cooking time as below;

|Recipe No.|Name|Material|Cookware|Procedure1|Procedure2|Procedure3|Procedure4|Procedure5|
---|---|---|---|---|---|---|---|---
|1|French Fries|Potato, Salad Oil|Toaster Oven|Peel & Cut:**5min**|set the material to the device:**1min**|(heating:**15min**)|clean up the sink(**1min**) ||
|2|Steamed Sweet Potatoes|Sweet Potato, Water|Electric Pot(*HEALSIO HOTCOOK KN-HW24C*)|Cut:**3min**|set the material to the device:**1min**|(heating:**15min**)|||
|3|Steamed Eggplant|Eggplant|Microwave|Cut:**4min**|set the material to the device:**1min**|(heating:**5min**)|||
|4|Miso Soup|Dried Sardines, Miso, Water, Onion|Electric Pot(*e-wonder OEDA30 3L*)|Peel & Cut:**3min**|set the material to the device:**2min**|(heating:**20min**)|||
|5|Grilled Fish|Fish|Grill|set the material to the device:**1min**|(burn the front:**5min**)|Turn over:**1min**|(burn the back:**3min**)|Extinguish:**1min**|
|6|Roast Beef|Beef, Sweet Sake, Soy Sauce|Electric Pot(*HEALSIO HOTCOOK KN-HW24C*)|make a seasoning:**2min**|heating seasoning:**2min**|set the material to the device:**2min**|(heating:**30min**)||
|7|Pancake|Pancake Mix, Egg, Milk|Frying Pan|preprocess material & set to device|(burn the front:**4min**)|Turn over:**1min**|(burn the back:**3min**)|Extinguish:**1min**|

* Though it takes much more time to cook roast beef, for computational simplexity, it is assumed that it itakes 30minute
* Though, after cooking, it is need to serve foods and clean them up, it is assumed that they doesn't be needed.
* After that, define indexes as below;

|Recipe No.|Name|Material|Cookware|Procedure1|Procedure2|Procedure3|Procedure4|Procedure5|
---|---|---|---|---|---|---|---|---
|1|French Fries|Potato, Salad Oil|Toaster Oven|***0***|***1***|-|***2***|-|
|2|Steamed Sweet Potatoes|Sweet Potato, Water|Electric Pot(*HEALSIO HOTCOOK KN-HW24C*)|***3***|***4***|-|-|-|
|3|Steamed Eggplant|Eggplant|Microwave|***5***|***6***|-|-|-|
|4|Miso Soup|Dried Sardines, Miso, Water, Onion|Electric Pot(*e-wonder OEDA30 3L*)|***7***|***8***|-|-|-|
|5|Grilled Fish|Fish|Grill|***9***|-|***10***|-|***11***|
|6|Roast Beef|Beef, Sweet Sake, Soy Sauce|Electric Pot(*HEALSIO HOTCOOK KN-HW24C*)|***12***|***13***|***14***|-|-|
|7|Pancake|Pancake Mix, Egg, Milk|Frying Pan|***15***|-|***16***|-|***17***|



## 3. Formulation
* Define the process time of i-th job as <img src="https://latex.codecogs.com/gif.latex?p_i" />
    * <img src="https://latex.codecogs.com/gif.latex?p_0=5,&space;p_1=1,&space;p_2=1,p_3=3,p_4=1,p_5=4,p_6=1,p_7=3,p_8=2,p_9=1,p_{10}=1,p_{11}=1,p_{12}=2,p_{13}=2,p_{14}=2,p_{15}=4,p_{16}=1,p_{17}=1" />

```python
p = {}
p[0] = 5
p[1] = 1
p[2] = 1

p[3] = 3
p[4] = 1

p[5] = 4
p[6] = 1

p[7] = 3
p[8] = 2

p[9] = 1
p[10] = 1
p[11] = 1

p[12] = 2
p[13] = 2
p[14] = 2

p[15] = 4
p[16] = 1
p[17] = 1
```

### 3-1. Define Variables

* Define <img src="https://latex.codecogs.com/gif.latex?x(j,k)" /> as a binary variable which equals to 1 when j-th procedure starts before k-th procedure and equals to 0 when job j doesn't starts before job k
* Define <img src="https://latex.codecogs.com/gif.latex?s(j)" /> as a continuous variable which explains the start time of j-th procedure

```python
# Deine Model
model = Model("Cooking")

# Define Variable
feature_num = len(p)

x = {}# if job j starts before job k => 1
for j in range(feature_num):
    for k in range(feature_num):
        if j != k:
            x[j, k] = model.addVar(vtype="B")
    
s = {}# job j's start time
for j in range(feature_num):
    s[j] = model.addVar(vtype="C")

model.update()
```

### 3-2. Add Constraint

#### 3-2-1. Symmetrical Constraint

* With the symmetrical constraint, <img src="https://latex.codecogs.com/gif.latex?x(j,k)&plus;x(k,j)=1&space;(\nabla&space;j&space;\neq&space;k)" /> can hold

```python
for j in range(feature_num):
    for k in range(feature_num):
        if j < k:
            model.addConstr( x[j, k] + x[k, j] == 1)
```
#### 3-2-2. Disjunctive Constraint

* By using [*BIG-M*](http://web.tuat.ac.jp/~miya/fujie_ORSJ.pdf) technique, constraint of starts time can be defined as below;

```python
# Disjunctive Constraint
for i in range(feature_num):
    for j in range(feature_num):
        if i != j:
            model.addConstr( s[i] + p[i] - s[j] <=     100*(1-x[i, j]))
```

#### 3-2-3. Start time Constraint

* For the constraint of the start time, below constraint can hold
    * <img src="https://latex.codecogs.com/gif.latex?\sum_{j&space;\neq&space;k}p(k)x(j,k)<=s(j)&space;(\nabla&space;j=0,1,...N-1)" />

```python
#Start Time Constraint
for j in range(feature_num):
    model.addConstr( quicksum(p[k]*x[k , j] for k in range(feature_num) if j != k)  <= s[j])
```

#### 3-2-4. Specific Constraint

* Consider below 4 specific constraint
    * In the same recipe, the job can't pass the post-job
    * When using grill or frying pan, for the safe, heating time is strictly obeyed
    * As *HOTCOOK* is shared with Steamed Sweet Potatoes and Roast Beef, so it can't use simultaneously
    * As Frying pan is shared with Roast Beef and Pancake, so it can't use simultaneously

```python
# In the same redipe, it can't pass post-job
list_ = [0, 1, 3, 5, 7, 12, 13]

for i in list_:
    model.addConstr( s[i] + p[i]  <= s[i+1] )

# when using grill or frying pan, for the safe, heating time is strictly obeyed
model.addConstr( s[9] + p[9] + 5 == s[10] )
model.addConstr( s[10] + p[10] + 3 == s[11] )

model.addConstr( s[15] + p[15] + 4 == s[16] )
model.addConstr( s[16] + p[16] + 3 == s[17] )

# As Hotcook is shared with Steamed Sweet Potatoes and Roast Beef, so it can't use simultaneously
model.addConstr(  s[4] + p[4] + 15 - s[14]   <=  100*(1-x[4, 14]) )
model.addConstr(  s[14] + p[14] + 30 - s[4]  <=  100*(1-x[14, 4]) )

# As Frying pan is shared with Roast Beef and Pancake, so it can't use simultaneously
model.addConstr(  s[13] + p[13] +  - s[17]   <=  100*(1-x[13, 17]) )
model.addConstr(  s[17] + p[17] +  - s[13]  <=  100*(1-x[17, 13]) )
```

### 3-3. Optimization

* Define objective function as below;
   * <img src="https://latex.codecogs.com/gif.latex?Minimize&space;\sum_{j=1}^{N}&space;s(j)" />

```python
model.setObjective(quicksum(s[j] for j in range(feature_num)), GRB.MINIMIZE )

model.optimize()
```

## 4. Result

### 4-1. Check the status and the result

|Status|Explain|
---|---
|1|Optimal|
|3|Infeasible|

```python
print(model.Status)

if model.Status == 1 :
    print('Opt. Value=', model.ObjVal)

    S_ = []
    for j in range(feature_num):
        S_.append([j, s[j].X])

dict_ = {"0":"0. French Fries:Peel&Cut", "1":"1. French Fries:set the material to the device", "2":"2. French Fries:clean up the sink", 
         "3":"3. Steamed Sweet Potatoes:Cut", "4":"4. Steamed Sweet Potatoes:set the material to the device",
         "5":"5. Steamed Eggplant:Cut", "6":"6. Steamed Eggplant:set the material to the device",
         "7":"7. Miso Soup:Peel&Cut", "8":"8. Miso Soup:set the material to the device", 
         "9":"9. Grilled Fish:set the material to the device", "10":"10. Grilled Fish:Turn over", "11":"11. Grilled Fish:Extinguish",
         "12":"12. Roast Beef:make a seasoning", "13":"13. Roast Beef:heating seasoning", "14":"14. Roast Beef:set the material to the device",
         "15":"15. Pancake:preprocess material & set to device", "16":"16. Pancake:Turn over", "17":"17. Pancake:Extinguish"}

result = pd.DataFrame( np.hstack( [np.array(S_)[np.argsort(np.array(S_)[:, 1]), :], 
                          np.array( [p[i] for i in  np.array(S_)[np.argsort(np.array(S_)[:, 1]), :][:, 0]] ).reshape(-1, 1)]), 
            columns=["Job Index", "Start Time", "Unit Time"])

result["Finish Time"] = result["Start Time"] + result["Unit Time"]
result["Job Name"] = result["Job Index"].astype("int").apply(lambda x : dict_[str(x)])
result[["Job Name", "Start Time", "Finish Time", "Unit Time"]]
```

|Index|Job Name|Start Time|Finish Time|Unit Time|
---|---|---|---|---
|0|3. Steamed Sweet Potatoes:Cut|0|3|3|
|1|12. Roast Beef:make a seasoning|3|5|2|
|2|15. Pancake:preprocess material & set to device|5|9|4|
|3|5. Steamed Eggplant:Cut|9|13|4|
|4|16. Pancake:Turn over|13|14|1|
|5|7. Miso Soup:Peel&Cut|14|17|3|
|6|17. Pancake:Extinguish|17|18|1|
|7|6. Steamed Eggplant:set the material to the de...|18|19|1|
|8|9. Grilled Fish:set the material to the device|19|20|1|
|9|0. French Fries:Peel&Cut|20|25|5|
|10|10. Grilled Fish:Turn over|25|26|1|
|11|11. Grilled Fish:Extinguish|29|30|1|
|12|4. Steamed Sweet Potatoes:set the material to ...|30|31|1|
|13|1. French Fries:set the material to the device|31|32|1|
|14|2. French Fries:clean up the sink|32|33|1|
|15|8. Miso Soup:set the material to the device|33|35|2|
|16|13. Roast Beef:heating seasoning|35|37|2|
|17|14. Roast Beef:set the material to the device|46|48|2|


### 4-2. Visualization

* Draw gantt chart

```python
color = []
for i in range(len(result)):
    if "French Fries" in result["Job Name"].iat[i]:
        color.append(0)
    elif "Steamed Sweet Potatoes" in result["Job Name"].iat[i]:
        color.append(1)
    elif "Steamed Eggplant" in result["Job Name"].iat[i]:
        color.append(2)
    elif "Miso Soup" in result["Job Name"].iat[i]:
        color.append(3)
    elif "Grilled Fish" in result["Job Name"].iat[i]:
        color.append(4)
    elif "Roast Beef" in result["Job Name"].iat[i]:
        color.append(5)
    elif "Pancake" in result["Job Name"].iat[i]:
        color.append(6)

result["Color"] = color

import plotly.express as px
import plotly.io as pio
from datetime import datetime

result["Start Time DT"] = result["Start Time"].apply(lambda x : datetime(2021, 2, 9, 0, 0, int(x)))
result["Finish Time DT"] = result["Finish Time"].apply(lambda x : datetime(2021, 2, 9, 0, 0, int(x)))

fig = px.timeline(result, x_start="Start Time DT", x_end="Finish Time DT", y="Job Name", color="Color")
fig.update_yaxes(autorange="reversed") # otherwise tasks are listed from the bottom up
pio.write_html(fig,file="./6-1.html")
```

{% include 6-1.html %}
