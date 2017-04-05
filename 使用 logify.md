<center> 使用 logify </center> 
----
#### 使用 logify

```
$ cd /Users/tomorrow/Desktop/Tweak/HOOK/hookapp
$ ls
Header					Tweak.xm		  
obj          			HooKApp.plist		
ViewController.h		packages
Makefile				control

$ /opt/theos/bin/logify.pl Header/LoginViewController.h > Tweak.xm
$ /opt/theos/bin/logify.pl Header/MainViewController.h >> Tweak.xm
$ /opt/theos/bin/logify.pl Header/RegisterViewController.h >> Tweak.xm
```
注：
> 1. `Header` 文件夹与 `Tweak.xm` 平级
> 2. 如果 `Tweak.xm` 文件中没有内容且是第一个文件时使用 `>` ，余下使用 `>>`
> 3. 如果 `Tweak.xm` 文件中有内容且是第一个文件时使用 `>>` ，余下使用 `>>`
> 4. `LoginViewController.h` 、`MainViewController.h` 和 `RegisterViewController.h` 此时必须在 `Header`文件夹中

#### Tweak.xm 文件修改

```
- (void).cxx_destruct { %log; %orig; }
- (NSString *)debugDescription { %log; NSString * r = %orig; HBLogDebug(@" = %@", r); return r; }
- (NSString *)description { %log; NSString * r = %orig; HBLogDebug(@" = %@", r); return r; }
- (unsigned long long )hash { %log; unsigned long long  r = %orig; HBLogDebug(@" = %llu", r); return r; }
- (Class )superclass { %log; Class  r = %orig; HBLogDebug(@" = %@", r); return r; }
- (void)dealloc { %log; %orig; }
- (void)didReceiveMemoryWarning { %log; %orig; }
```
这些方法直接删除  

#### ERROR 修改
> error: unknown type name 'CDUnknownBlockType'  
> error: unknown type name 'WebViewJavascriptBridge'  
> error: unknown type name 'OrderListModel'  
> error: '(unsign int)'  
> error: 'NSString-xxxx'  

1. `CDUnknownBlockType` 全局替换成 `id`
2. 如果方法用不到直接删除，如果必须用到就在 `ViewController.h` 文件中添加属性
3. `(unsign int)` 直接全局删除，输出换成 `%@` 或 `%d`
4. `%hook NSString` 中数据包含了 `%hook NSString-xxxx`

添加属性：

```
@interface OrderListModel : NSObject

@property(retain, nonatomic) NSMutableArray *ordersArray;
@property(copy, nonatomic) NSString *recommend_tag;
@property(copy, nonatomic) NSString *refreshId;
@property(retain, nonatomic) NSArray *tags;
@property(copy, nonatomic) NSString *task_id;
@property(retain, nonatomic) NSDictionary *task_order_over_time_allowance;
@property(copy, nonatomic) NSString *time_limit;
@property(copy, nonatomic) NSString *type;

@end

@interface STHomeOrderListViewController : UIViewController

@property(retain, nonatomic) OrderListModel *currentOrderListModel;
@property(retain, nonatomic) OrderListModel *detailMapTask;

@end
```


