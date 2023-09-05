### Basics
Key Value Pair
```yaml
# [key]: [value]
Fruit: Apple
Vegetable: Carrot
```

Array/List
```yml
Fruits:
- Orange
- Apple
- Banana

Vegetables:
- Carrot
- Cauliflower
- Tomato
```

Dictionary/Map
```yaml
Banana:
	Calories: 105
	Fat: 0.4 g
	Carbs: 27 g

Grapes:
	Calories: 62
	Fat: 0.3 g
	Carbs: 16 g
```

The number of spaces in YAML is important if you were to add a space before "Fat" and "Carbs" you'll get an syntax error as they are now Properties of "Calories" which is not possible as it has a defined value of "105"
```yaml
Banana:
	Calories: 105
		Fat: 0.4g
		Carbs: 27g
```

List of Dictionaries
```yaml
- Color: Blue
  Model:
	  Name: Corvette
	  Model: 1995
  Transmission: Manual
  Price: $20000

- Color: Red
  Model:
	  Name: Corvette
	  Model: 1995
  Transmission: Automatic
  Price: $20000
  
- Color: Green
  Model:
	  Name: Corvette
	  Model: 1995
  Transmission: Manual
  Price: $20000
```

Properties can be ordered in any order within a dictionary and they remain the same. In an Array or List the order matters

Putting a # in front of a line turns it into a comment

### Pods with YAML
Configuring pod properties with yaml
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: frontend
spec:
  containers:
    - name: nginx-container
      image: nginx
```

To create the pod we run following command while referencing the yml
```bash
kucectl create -f pod-definition.yml
```

to find details of a pod write
```bash
kubectl describe pod [podname]
```

Tip: Support for Kubernetes and YAML formatting in VS Code
