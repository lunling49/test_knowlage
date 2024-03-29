##### APP崩溃的情况：闪退、提示停止运行、无响应

不同情况虽然没有严格意义上区分开引起原因，但是都有侧重

##### 1、接口返回值

直接原因：

- app无法解析接口返回值
- 获取不到要获取的参数
- 参数类型不对 导致客户端代码报错

引起原因：

- 脏数据
- 网络问题导致接口超时或漏了数组元素
- 前后台没有统一参数类型标准
- 参数名错误
- 实体消失 

解决办法：

在网络顺畅/不顺畅情况下抓包，对着api文档一个一个的参数对比，返回值有数组可以横向对比，可能是其中某个元素内的某个参数和其他元素内的这个参数有内容不同/类型不同／为空/不存在/规范不同

测试方法：

- 首先要从2个角度考虑。1：后台不要返回这种脏数据，或者有脏数据要进行处理再返回给app。2：app要有一定的容错性，不能因为一个参数这么一点小事就导致崩溃（低级bug瞬间升级到致命bug）。所以要从俩边测试。1：先进行正常的接口测试，保证正常数据返回没有问题。再通过操作数据库或其他手段进行构造脏数据，测试服务器的错误处理能力。2：再利用mock或抓包工具，强行修改返回值，测试app端的容错能力。用脚本或手动把所有/特定 的参数进行更改，包括 类型/内容长度／为空／删除掉／不符合规范 等情况来测试app的容错性和成熟性。
- 其次网络问题也是有概率引起崩溃，就是在网络环境很恶劣或变动频繁的情况下进行所有接口测试，保证返回值全面完整。观察接口返回是否有拉下的数组元素。因为app的超时判定和服务器的超时判定是不统一的。可能接口超时要60秒，但是app只等待10秒钟，10秒没到就判定失败了，但这不是导致崩溃的原因。导致崩溃的原因在于服务器返回超时后（不是无网络，不是关掉wifi或数据流量），接口报什么http状态码，一般是502，app原则上是要对所有接口502都有对应处理和提示，但实际情况是，很多接口有提示不崩溃，更多的接口会崩溃。所以测试的时候要构造特殊环境，来让所以接口依次超时。方法可以是在抓包工具上打断点，然后不进行继续操作，挺着看app最终会不会崩溃。
-  实体消失问题导致崩溃，其实是接口规范上的原因，当因为先后操作，页面未及时刷新的情况，导致app对一个已经在后台数据库抹除的实体或关系进行访问时，后台又恰好没考虑过此情况，导致后台返回结果不可预料，app又没有抓取某种异常返回，导致崩溃。测试办法就是测试点中计划好所有这种可以操作到消失实体的情况，来进行模拟测试。或者抓包时强行更改请求实体，来达到请求一个不存在实体的场景，观察服务器如何处理并返回，app又是否会因此而崩溃。



##### 2、内存问题

直接原因：客户端APP代码报错

引起原因：

- 兼容不好
- 内存不足
- 内存泄露造成app开辟内存空间失败
- 内存泄漏

解决办法：

提醒用户更换手机或关掉后台其他app进程，崩溃的app要进行全面测试，定位到具体什么操作导致崩溃。

测试方法：

先进行兼容性测试，用不同的操作系统/手机型号／品牌／系统版本／蓝牙版本去执行一些跟写入读取有关的功能的用例。用emmagee监控app，看到各种操作后，占用的内存是否超过预期。让开发规范代码，及时释放掉占用的存储空间。手机安装很多app，然后后台都打开，然后再运行自家app，观察其是否会崩溃频繁，可以用monkey测试（虽然monkey无法表明到底是什么原因引起崩溃，但是可以通过 观察后台干净／后台运行过多app 这俩种情况下多次测试，看是否因为后台运行过多app 就导致monkey崩溃概率高。而判断出大致自家app的生存能力）其他待补充。



##### 3、下标越界问题

直接原因：客户端APP代码报错

引起原因：

- 需要操作的元素已经消失
- 代码错误，超出实体数量
- 读取or写入本地文件或缓存时的IO错误

解决办法：调查引起崩溃的具体操作步骤，然后提交开发解决，前端代码容错率需要提高。

测试方法：

- 边界值测试为核心思想，测试正常情况有关数量的功能用例
- 要进行代码review。1：保证代码没有错误，循环中没有超出实体数量。2：保证代码容错性高，每个循环都要有越界异常捕获并处理。
- 要进行手动破坏性测试。1：如删除本地文件，比如app要调取本地缓存的4张图片，在app刚要调用的时候，已经选择好的时候，切换到本地文件管理中，删掉其中一个，那么app就会访问到一个不存在的文件，会引发越界等代码报错。2：破坏掉这个文件。那么app就会读取的时候发生io错误。等情况来进行测试。




##### 4、渲染不及时问题

直接原因：控件生成/调用受阻，导致前端app代码报错

引起原因：渲染过慢，操作过快，兼容性不好

解决办法：

让用户换手机，或慢点点，重新设计避免用户连点造成的操作过快，重新设计减轻页面加载渲染负担，异步处理

测试方法：

对复杂／卡顿页面进行快速操作来让本不应该出现在一起的俩个控件出现在一起，或用monkey最大速度测试



##### 5、权限问题

直接原因：客户端未对无权限情况处理，导致代码报错

引起原因：用户访问未获取到系统相关权限的功能，客户端又未对此情况进行处理

解决办法：修改崩溃bug，设计此情况的处理机制，如提示用户去手动开权限，或自动退出等情况。

测试方法：

关掉app所有的系统权限，然后去访问所有系统权限相关的页面和功能。例如：相册，照相，定位，开启wifi，蓝牙，gps 等等权限。



##### 6、第三方问题

引起原因：

- 第三方广告的突然弹出
- 其他app分享进来和出去
- 各种第三方app的强行抢镜（如抢红包提醒）

测试方法：

在各个页面，手动触发大多数app的或本app的外接广告来测试。用其他主流app测试分享，或自家app分享出去再回来看是否已经被退出。突然收到其他app的强制提醒。



##### 7、系统高优先级APP问题

直接原因：导致自家app突然被挂起或放置后台

引起原因：突然来电话，突然收短信，闹钟，会议提醒系统原生app等情况

测试方法：

在各个页面，功能运行前中后。进行接电话／短信来测试。主要测试是否会影响电话/短信，电话/短信结束后 app是否能恢复到之前的页面,还是已经闪退被强关了。



##### 8、设备视图方向问题

直接原因：因横竖屏导致app崩溃

解决方法：重启APP

测试方法：

- 先横，再开app
- 先竖，再开app
- 开app后，各种页面上，功能前中后，横屏／竖屏来回切换



##### 9、多语言问题

直接原因：各种语言导致崩溃

测试方法：

- 先切换成各国语言，再开app进行各种功能用例测试
- 先开app，再来回切换各国语言进行测试



##### 10、其他代码错误

直接原因：客户端APP代码错误

引起原因：各种异常操作，正常操作

解决办法：adb shell logcat抓日志，后台查看崩溃日志

测试方法：执行全部测试用例即可



##### 11、弱网问题

直接原因：客户端无法解析json返回值

引起原因：网络差，json串过长

解决办法：提醒用户换更快网络，客户端对此操作增加等待时间。接口返回进行异步处理。增加翻页功能。

测试方法：用抓包工具模拟出弱网环境，包含丢包率，稳定性等元素。然后对接口返回值构造超长数据进行测试。