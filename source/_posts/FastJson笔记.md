---
title: FastJson笔记
date: 2019-05-28 16:20:19
tags:
- JSON
- SpringBoot
---
## Maven依赖引入
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>x.x.x</version>
    </dependency>


## 注解使用
### JSONField使用

    @JSONField(name="AGE", serialize=false)
    private int age;
    
    @JSONField(name="LAST NAME", ordinal = 2)
    private String lastName;
    
    @JSONField(name="FIRST NAME", ordinal = 1)
    private String firstName;
    
    @JSONField(name="DATE OF BIRTH", format="dd/MM/yyyy", ordinal = 3)
    private Date dateOfBirth;

1. name:jsonKey重命名
2. serialize:不进行序列化，也就是不在json返回值中进行输出
3. ordinal:指定了输出顺序
4. format:格式化date属性

## 输出
### JAVA转JSON对象

    String jsonOutput= JSON.toJSONString(listOfPersons);

### BeanToArray
对bean进行排序

    String jsonOutput= JSON.toJSONString(listOfPersons,SerializerFeature.BeanToArray);

### 创建JSON对象
new一个JSONObject或者JSONArray就可以

### JSON转JAVA对象

    Person newPerson = JSON.parseObject(jsonObject, Person.class);

    备注：若bean中指定字段中有 `@JSONField(name = "DATE OF BIRTH", deserialize=false)` deserialize则不会被反序列化

### 使用ContextValueFilter作Value过滤

    @Test
    public void givenContextFilter_whenJavaObject_thanJsonCorrect() {
        ContextValueFilter valueFilter = new ContextValueFilter () {
            public Object process(
            BeanContext context, Object object, String name, Object value) {
                if (name.equals("DATE OF BIRTH")) {
                    return "NOT TO DISCLOSE";
                }
                if (value.equals("John")) {
                    return ((String) value).toUpperCase();
                } else {
                    return null;
                }
            }
        };
        String jsonOutput = JSON.toJSONString(listOfPersons, valueFilter);
    }