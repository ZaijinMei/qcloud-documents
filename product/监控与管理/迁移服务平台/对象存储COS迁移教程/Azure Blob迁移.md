

##  操作场景
下文将详细介绍当源对象存储部署在 AZURE Blob 时，如何配置全托管迁移任务和半托管迁移任务，实现数据迁移。


## 准备工作

#### AZURE存储 Blob
5.	您可以在Azure控制台，单击**存储账户**，然后选择待迁移的存储账户。
5.	在待迁移存储账户页面左侧导航栏单击**访问密钥**，查看密钥和连接字符串。

#### 腾讯云对象存储 COS
1. 创建目标存储空间，用于存放迁移的数据，详情请参见 [创建存储桶](https://cloud.tencent.com/document/product/436/13309)。
2. 创建用于迁移的子用户并授予相关权限：
	1.	登录腾讯云 [访问管理控制台](https://console.cloud.tencent.com/cam/overview)。
	2.	在左导航栏中选择**用户** > **用户列表**，进入用户列表页面。
	3.	新建子用户，勾选编程访问及腾讯云控制台访问。
	4.	搜索并勾选 QcloudCOSAccessForMSPRole 及 QcloudCOSFullAccess 策略。
	5.	完成子用户创建并保存子用户名，访问登录密码，SecretId，SecretKey。


>?迁移服务也可以使用主账号操作，但是出于安全考虑，建议新建子账号并使用子账号API密钥进行迁移，迁移完成后删除。

## 操作步骤

#### 登录迁移服务平台

1. 登录 [迁移服务平台](https://console.cloud.tencent.com/msp)。
2. 在左导航栏中单击**对象存储迁移**，进入对象存储迁移页面。


#### 新建迁移任务

1. 在对象存储迁移页面，单击**新建任务**，进入对象存储迁移任务配置页面，进行迁移参数的设置。

2. 设置迁移任务名称。
   ![](https://main.qcloudimg.com/raw/5094fb4a8b2e20e3400c9388d6eeeadb.jpg)
   任务名称：字符长度为1至60个字符，允许的字符为中文、英文、0-9、\_、-。此处设置的名称，将用于在任务列表中查看迁移状态和迁移进度。

3. 预估任务规模。

   ![](https://qcloudimg.tencent-cloud.cn/raw/f97b88b85b49b4565dfd30f1f251d881.png)

   更准确的填写任务规模，以便我们更好的准备相关资源，非必填

4. 设置要迁移的文件来源。
   此处迁移源服务提供商应选择`AZURE`，并在下方 存储账户，密钥文本框中输入可用于迁移的账号 存储账号和 密钥。接着输入待迁移容器名称和连接字符串，其中连接字符串不输入的情况下默认为core.windows.net。
   ![](https://qcloudimg.tencent-cloud.cn/raw/6f29a41f65fed629bf7b22947be357a7.png)

5. 选择 Header 方式。
   如果源桶中的文件设定了 Header/Tag 并且需要在迁移后保留，请选择保留或设置替换规则。
   ![](https://main.qcloudimg.com/raw/2673cf039f0fa4c36651ddb63dff171d.jpg)

6. 设定迁移规则。
   选择对指定桶中的全部文件进行迁移，或仅迁移指定前缀的文件。
   ![](https://main.qcloudimg.com/raw/50d5e4c411e5a31dd4f0969c9fe7f0c4.jpg)

7. 设定时间范围。
   开启时间范围，只迁移指定时间范围内新增或变更的文件。
   ![](https://main.qcloudimg.com/raw/94fce7cc7c9b2402d0e28babc0050e9a.jpg)

8. 选择文件存储方式。
   根据迁移的需求，设定迁移后文件的存储方式，可以选择：标准存储、低频存储、保持原存储属性、归档存储。
    ![](https://qcloudimg.tencent-cloud.cn/raw/65f77fa2c28058f8b704991c0afee014.png)

9. 设定执行速度。
   各公有云厂商的对象存储都有速度限制。为确保业务稳定，请在迁移前与源厂商确认并设置最高迁移可用 Mbps。
   ![](https://qcloudimg.tencent-cloud.cn/raw/1a50bcfbfe4dfc0a6487f4d8834b1acb.png)

10. 选择要迁移到的目标位置。
    在迁移目标信息中，输入用于迁移的腾讯云子用户 SecretId，SecretKey。填入密钥后，单击“迁移桶名称”下拉框右侧的**刷新**按钮，即可获取目标对象存储桶列表。
    ![](https://main.qcloudimg.com/raw/0b92939c9a50636395da7e4713dbded9.jpg)

11. 指定迁移到目标桶的指定目录和同名文件处理方式。

     - 保存到根目录： 直接将源桶中的文件按原始相对路径保存到目标桶的根目录。
     - 保存到指定目录：将源桶中的文件保持原始相对路径保存到指定目录中。
       ![](https://main.qcloudimg.com/raw/d69721ab8a5b10bf0b43976a5db265d3.jpg)
       例如：
       源桶中的文件`/a.txt`，`/dir/b.txt`两个文件，文本框中填写“dest”，那么迁移后这两个文件在目标桶中的路径为：`/dest/a.txt`，`/dest/dir/b.txt`。
       如果文本框中填写`dest/20180901`，那么迁移后这两个文件在目标桶中的路径为：`/dest/20180901/a.txt`，`/dest/20180901/dir/b.txt`。

    >!
    >
    >- 若同名文件选择**覆盖**，则迁移时会直接覆盖掉同名文件
    >- 若同名文件选择**跳过**，会基于文件最后修改时 LastModified 判断，即
    >  - 如果源地址中文件的 LastModified 晚于或者等于目的地址中文件的 LastModified，则执行覆盖。
    >  - 如果源地址中文件的 LastModified 早于目的地址中文件的 LastModified，则执行跳过。
    >- 若在迁移过程中对象（文件）内容有变化，需要进行二次迁移。

    

12. 选择迁移模式

     - 新建迁移任务后立即启动全托管迁移：选择托管迁移，用户单击“新建并启动”后 MSP 服务将通过公网访问源存储进行迁移。

       ![](https://qcloudimg.tencent-cloud.cn/raw/9c9591c252be2d19c67ca994dc745c23.png)

     - 新建迁移任务后手动下载 Agent 启动迁移：选择 Agent 模式迁移，用户在单击“新建并启动”后，将仅创建任务配置，需要用户手动下载 Agent 在迁移源一侧的服务器上部署之后才会正式启动迁移。迁移 Agent 部署参考文档：[半托管迁移 Agent 的使用说明](https://cloud.tencent.com/document/product/659/81158)。
       ![](https://qcloudimg.tencent-cloud.cn/raw/1cf17dbd3078237e54d5d878c2af73e1.png)

    

13. 定时任务设置

    定时任务可以重复运行任务，可以对源桶中的增量文件做出同步。除了第一次任务是立即执行，之后会根据定时设置在固定间隔时间之后或者 Cron 设定的定时规则时触发再次运行。再次运行时“同名文件”会变为“跳过（保留目标桶中已有的同名文件）”，即只同步增量文件。

    ![](https://qcloudimg.tencent-cloud.cn/raw/3c222b9c970209607b81e0fc40d48118.png)

    

14. 单击**新建并启动**，即可启动迁移任务。

![](https://qcloudimg.tencent-cloud.cn/raw/5dabf24cdbbef293f85c7fa10c4816de.png)


## 查看迁移状态和进度

在文件迁移工具主界面中，可以查看所有文件迁移任务的状态和进度：
•	“任务完成”状态，绿色是任务完成并且所有文件都迁移成功，黄色是迁移任务完成但部分文件迁移失败。
•	单击“重试失败任务”链接后，该任务中失败的文件将会重试迁移，已经成功迁移的文件不会重传。
•	单击“导出”链接可以导出迁移过程中失败的文件列表。


## 预估文件迁移时间

迁移速度由迁移过程中涉及到的每一个环节的最低速度决定，同时受到网络传输速度和最大并发数影响。影响因素有：

| 影响因素               | 说明                                                         |
| ---------------------- | ------------------------------------------------------------ |
| 迁出源的读取速度       | 数据源的读取速度因不同的服务商而不同，通常：<br><li>传输速度在50Mbps - 20000Mbps之间。<br><li>文件读取QPS在500 - 5000之间（大量小文件的传输受并发限制）。 |
| MSP 平台的传输速度     | MSP 平台全托管模式下提供最大2000Mbps的迁移带宽；半托管模式下取决于客户提供的迁移资源大小 |
| 迁入目标位置的写入速度 | 腾讯云对象存储 COS：默认写入传输速度15000Mbps，写入QPS 3000 - 30000之间。 |



