---
title: Django模型的Field Types总结
tags:
  - Django
  - Web框架
categories: python学习心得&备忘
abbrlink: 62657fb0
date: 2017-08-07 13:25:10
---

** Field Types **
常用参数：
** null **
如果设置为 True , Django 存放一个 NULL 到数据库字段。默认为 False。
** blank **
如果设置为 True , 此 field 允许为 blank （空白），默认为 False。
** choices **
一个2元元组的元组或者列表，如果执行 choices ， Django 的 admin 就会使用 选择框而不是标准的 text 框填写这个 field。
```
YEAR_IN_SCHOOL_CHOICES = (
    (u'FR', u'Freshman'),
    (u'SO', u'Sophomore'),
    (u'JR', u'Junior'),
    (u'SR', u'Senior'),
    (u'GR', u'Graduate'),
)
```
<!-- more -->
2元元组的第一个元素是要存入 database 的数据，第二个元素是 admin 的界面 显示的数据。 
使用了 choices 参数的 field 在其 model 示例里，可以用 "get_field的名 字_display" 方法 显示 choices 的显示字串（就是2元元组的第二个数据）。示 例：
```
from django.db import models

class Person(models.Model):
    GENDER_CHOICES = (
        (u'M', u'Male'),
        (u'F', u'Female'),
    )
    name = models.CharField(max_length=60)
    gender = models.CharField(max_length=2, choices=GENDER_CHOICES)
```

```
>>> p = Person(name="Fred Flinstone", gender="M")
>>> p.save()
>>> p.gender
u'M'
>>> p.get_gender_display()
u'Male'
```

** default **
field 的默认值，可以使用可调用对象（a callable object），如果使用可调用 对象，那么每次创建此 model 的新对象时调用可调用对象。常见如 datatime 。
** help_text **
help_text 的值可以在 admin form 里显示，不过即使不使用 admin ，也可以当 做描述文档使用。
** primary_key **
如果为 True ， 这个 field 就是此 model 的 primary key 。
** unique **
如果为 True， 此 field 在这个 table 里必须唯一。
** verbose_name **
verbose，详细的意思。verbose_name，就可以理解为详细的名字吧。
除了ForeignKey, ManyToManyField 和 OneToOneField之外，每个类型的字段都有一个可选的第一位置参数－详细的名字。如果没有给出详细的名称，Django将自动使用字段的属性名来代替他。替代过程中会转换下划线为空格。
该字段中，名字的详情为”person’s first name”：
first_name = models.CharField("person's first name", max_length=30)

以下字段中，first_name的详细名字为"first name":
first_name = models.CharField(max_length=30)

ForeignKey, ManyToManyField 和 OneToOneField要求第一个参数是模型的类，所以需要使用verbose_name关键字参数，如：
poll = models.ForeignKey(Poll, verbose_name="the related poll")
sites = models.ManyToManyField(Site, verbose_name="list of sites")
place = models.OneToOneField(Place, verbose_name="related place")

在需要的时候Django会自动大写 verbose_name的首字母。
原来verbose_name字段就是为ForeignKey, ManyToManyField 和 OneToOneField这三种关系准备的啊！

## 常见Filed Types
** 1、AutoField **
如果没有指明主键，就会产生一个自增的主键。
** 2、BigIntegerField **
64位的整型数值，从 -2^63 (-9223372036854775808) 到 2^63-1(9223372036854775807)。
** 3、BinaryField **
存储原始二进制数据，仅支持字节分配。功能有限。
** 4、BooleanField **
布尔型和NullBooleanField有区别，true/false，本类型不允许出现null。
** 5、CharField **字符串，一般都在创建时写入max_length参数。
** 6、CommaSeparatedIntegerField **
逗号分隔的整数，考虑到数据库的移植性，max_length参数应该必选。
原文解释：A field of integers separated by commas. As in CharField, the max_length argument is required and the note about database portability mentioned there should be heeded.
** 7、DateField **
时间，对应Python的datetime.date，额外的参数：DateField.auto_now表示是否每次修改时改变时间，DateField.auto_now_add 表示是否创建时表示时间，一般来说数据库重要的表都要有这样的字段记录创建字段时间个最后一次改变的时间。关于时间的话，建议timestamp，当然 python的话还是DateTime吧。
** 8、DateTimeField **
对应Python的datetime.datetime，参照参数（7）。
** 9、DecimalField **
固定精度的十进制数，一般用来存金额相关的数据。对应python的Decimal，额外的参数包括DecimalField.max_digits和DecimalField.decimal_places ，这个还是要参照一下mysql的Decimal类型，[http://database.51cto.com/art/201005/201651.htm](http://database.51cto.com/art/201005/201651.htm)
例如：price = models.DecimalField(max_digits=8,decimal_places=2)
** 10、EmailField **
字符串，会检查是否是合法的email地址
** 11、FileField **
class FileField([upload_to=None, max_length=100, **options])
存文件的，参数upload_to在1.7之前的一些老版本中必选的
** 12、FloatField **
浮点数，必填参数：max_digits，数字长度；decimal_places，有效位数。
** 13、ImageField **
class ImageField([upload_to=None, height_field=None, width_field=None, max_length=100, **options])
图片文件类型，继承了FileField的所有属性和方法。参数除upload_to外，还有height_field，width_field等属性。
** 14、IntegerField **
[-2147483648,2147483647 ]的取值范围对Django所支持的数据库都是安全的。
** 15、IPAddressField **
点分十进制表示的IP地址，如10.0.0.1
** 16、GenericIPAddressField **
ip v4和ip v6地址表示，ipv6遵循RFC 4291section 2.2,
**17、NullBooleanField**
可以包含空值的布尔类型，相当于设置了null=True的BooleanField。
**18、PositiveIntegerField**
正整数或0类型，取值范围为[0 ,2147483647]
**19、PositiveSmallIntegerField**
正短整数或0类型，类似于PositiveIntegerField，取值范围依赖于数据库特性，[0 ,32767]的取值范围对Django所支持的数据库都是安全的。
**20、SlugField**
只能包含字母，数字，下划线和连字符的字符串，通常被用于URLs表示。可选参数max_length=50，prepopulate_from用于指示在admin表单中的可选值。db_index，默认为True。
**21、SmallIntegerField**
小整数字段，类似于IntegerField，取值范围依赖于数据库特性，[-32768 ,32767]的取值范围对Django所支持的数据库都是安全的。
**22、TextField**
文本类型
**23、TimeField**
时间，对应Python的datetime.time
**24、URLField**
存储URL的字符串，默认长度200；verify_exists(True)，检查URL可用性。
**25、FilePathField**

class FilePathField(path=None[, match=None, recursive=False, max_length=100, **options])
类似于CharField，但是取值被限制为指定路径内的文件名，path参数是必选的。
