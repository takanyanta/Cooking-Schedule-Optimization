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
|2|Steamed Sweet Potatoes|Sweet Potato, Water|Electric Pot(*HEALSIO HOTCOOK KN-HW24C*)|Cut(**3min**)|set the material to the device:**1min**|(heating:**15min**)|||
|3|Steamed Eggplant|Eggplant|Microwave|Cut(**4min**)|set the material to the device:**1min**|(heating:**5min**)|||
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
* Define the process time of i-th procedure as <img src="https://latex.codecogs.com/gif.latex?p_i" />

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

* Define <img src="https://latex.codecogs.com/gif.latex?x(j,k)" /> as a binary variable which equals to 1 when j-th procedure starts before k-th procedure and equals to 0 when procedure j doesn't starts before procedure k
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


