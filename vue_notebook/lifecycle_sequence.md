# 生命周期钩子执行顺序

```
		App
	/		\
	A		 B
	/\
A-1		A-2
```
App mount 之后销毁 App，打印所有生命周期钩子顺序

```
before create A 
create A 
before mount A 
	
	before create A-1 
	create A-1 
	before mount A-1 

	before create A-2 
	create A-2 
	before mount A-2 

before create B 
create B 

before mount B 

mount A-1 
mount A-2 
mount A 
mount B 

before destroy A 
	before destroy A-1 
	destroy A-1 
	
	before destroy A-2 
	destroy A-2 
destroy A 

before destroy  B 
destroy  B 
```
