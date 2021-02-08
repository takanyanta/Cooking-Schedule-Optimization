# Cooking-Optimization
Linear Programming, Optimization, Production Engineering(Industrial Engineering)

## Purpose

* To explore optimal procedure of cooking
* To minimize cumulative cooking time

## Assumption

* Define cooking time as below;

|Recipe No.|Name|Material|Cookware|Procedure1|Procedure2|Procedure3|Procedure4|Procedure5|
---|---|---|---|---|---|---|---|---
|1|French Fries|Potato, Salad Oil|Toaster Oven|Peel & Cut:**5min**|set the material to the device:**1min**|(heating:**15min**)|clean up the sink(**1min**) ||
|2|Steamed Sweet Potatoes|Sweet Potato, Water|Electric Pot(*HEALSIO HOTCOOK KN-HW24C*)|Cut:**3min**|set the material to the device:**1min**|(heating:**15min**)|||
|3|Steamed Eggplant|Eggplant|Microwave|Cut:**4min**|set the material to the device:**1min**|(heating:**5min**)|||
|4|Miso Soup|Dried Sardines, Miso, Water, Onion|Electric Pot(*e-wonder OEDA30 3L*)|Peel & Cut:**3min**|set the material to the device:**2min**|(heating:**20min**)|||
|5|Grilled Fish|Fish|Grill|set the material to the device:**1min**|(burn the front:**5min**)|Turn over:**1min**|(burn the back:**3min**)|Extinguish:**1min**|
|6|Roast Beef|Beef, Sweet Sake, Soy Sauce|Electric Pot(*HEALSIO HOTCOOK KN-HW24C*)|make a seasoning:**2min**|heating seasoning:**2min**|set the material to the device:**2min**|(heating:**30min**)||
|7|Pancake|Pancake Mix, Egg, Milk|Frying Pan|preprocess material|(burn the front:**4min**)|Turn over:**1min**|(burn the back:**3min**)|Extinguish:**1min**|

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



## Formulation
* Define the process time of i-th job as <img src="https://latex.codecogs.com/gif.latex?p_i" />

(<img src="https://latex.codecogs.com/gif.latex?p_0=5,&space;p_1=1,&space;p_2=1,p_3=3,p_4=1,p_5=4,p_6=1,p_7=3,p_8=2,p_9=1,p_{10}=1,p_{11}=1,p_{12}=2,p_{13}=2,p_{14}=2,p_{15}=4,p_{16}=1,p_{17}=1" />)

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

### Define Variables

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

### Add Constraint

#### 1. Symmetrical Constraint

With the symmetrical constraint, <img src="https://latex.codecogs.com/gif.latex?x(j,k)&plus;x(k,j)=1&space;(\nabla&space;j&space;\neq&space;k)" /> can hold

```python
for j in range(feature_num):
    for k in range(feature_num):
        if j < k:
            model.addConstr( x[j, k] + x[k, j] == 1)
```
### 2. Disjunctive Constraint

* By using [*BIG-M*](http://web.tuat.ac.jp/~miya/fujie_ORSJ.pdf) technique, 
If job j starts before job k, 

```python
# Disjunctive Constraint
for i in range(feature_num):
    for j in range(feature_num):
        if i != j:
            model.addConstr( s[i] + p[i] - s[j] <=     100*(1-x[i, j]))
```

#### 3. Start time Constraint

For the constraint of the start time, below constraint can hold

<img src="https://latex.codecogs.com/gif.latex?\sum_{j&space;\neq&space;k}p(k)x(j,k)<=s(j)&space;(\nabla&space;j=1,2,...J)" />

```python
#Start Timeに対する制約
for j in range(feature_num):
    model.addConstr( quicksum(p[k]*x[k , j] for k in range(feature_num) if j != k)  <= s[j])
```

#### 4. Specific Constraint


```python
list_ = [0, 1, 3, 5, 7, 12, 13]

for i in list_:
    model.addConstr( s[i] + p[i]  <= s[i+1] )

model.addConstr( s[9] + p[9] + 5 == s[10] )
model.addConstr( s[10] + p[10] + 3 == s[11] )

model.addConstr( s[15] + p[15] + 4 == s[16] )
model.addConstr( s[16] + p[16] + 3 == s[17] )

model.addConstr(  s[4] + p[4] + 15 - s[14]      <=              100*(1-x[4, 14]) )
model.addConstr(  s[14] + p[14] + 30 - s[4]      <=              100*(1-x[14, 4]) )
```

## Optimization

```python
model.setObjective(quicksum(s[j] for j in range(feature_num)), GRB.MINIMIZE )

model.optimize()
```
