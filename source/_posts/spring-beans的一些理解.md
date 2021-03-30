---
title: spring-beans的一些理解
date: 2021-03-05 13:12:02
tags: 
- JAVA
- Spring
---

## SimpleAliasRegistry

simpleAliasRegistry实现了aliasRegistry，aliasRegistry定义了对benas的alias的一些增删改查操作。

- public void registerAlias(String name, String alias)

  - 判断入参是否为空
  - 锁定aliasMap
    - name与alias是否相同；
    - 判断是否可能出现不同alias对应同一个name的情况；
    - 根据allowAliasOverriding决定是否存入这一对键值对；
    - 通过以下代码判断，来避免出现同一个alias不同class的问题；

    ```java
    public boolean hasAlias(String name, String alias) {
        for (Map.Entry<String, String> entry : this.aliasMap.entrySet()) {
          String registeredName = entry.getValue();
          if (registeredName.equals(name)) {
            String registeredAlias = entry.getKey();
            if (registeredAlias.equals(alias) || hasAlias(registeredAlias, alias)) {
              return true;
            }
          }
        }
        return false;
      }
    ```
    
    - 最后使用了put方法塞入。
    ```java
    this.aliasMap.put(alias, name);
    ```
    

`由此可以得出，rigisterAlias在配置允许（即在对其方法进行复写后）的情况下可以存在同一个class对应不同的alias，但是绝对不允许出现不同的class对应同一个alias`

- public void removeAlias(String alias)

  使用map的remove方法尝试移除alias，但是若返回定name为null则表示不存在其alias

  ```java
  String name = this.aliasMap.remove(alias);
  ```

- public boolean isAlias(String name)

  利用map的contain方法判断该class是否存在别名
  ```java
  this.aliasMap.containsKey(name);
  ```

- public String[] getAliases(String name)

由于在注册的时候说过在特殊情况下允许一个class对于多个alias，因此返回的可能是多个alias。

```java
private void retrieveAliases(String name, List<String> result) {
		this.aliasMap.forEach((alias, registeredName) -> {
			if (registeredName.equals(name)) {
				result.add(alias);
				retrieveAliases(alias, result);
			}
		});
	}
```

其中在判断过程中使用到了递归，利用alias与registeredName进行比较，为了避免map别名套别名的情况。例如：

```java
this.aliasMap.put("char1","java.lang.Char");
this.aliasMap.put("byte","char1");
```


