

# 任务汇总

---

- [ ] 禅道bug

- [ ] 环境数据：9/10







---

# 2025/3

## 23

#### 开发上传预警底图资料

```cpp
//update_db.sql
CREATE TABLE IF NOT EXISTS e_warning_map(
`id` TINYINT AUTO_INCREMENT PRIMARY KEY,
`filename` VARCHAR(100) DEFAULT NULL COMMENT '上传的预警底图文件名'
)ENGINE = InnoDB AUTO_INCREMENT=1 CHARACTER SET = utf8;

//router.cpp
Routes::Post(RoutesManager::router, "/upload_save_warningImg/:name",
               Routes::bind(&cmd::uploadStoreWarningImage));//上传图片并入库

//commands.cpp
void uploadStoreWarningImage(const Rest::Request &request, Http::ResponseWriter response) 
{
  response.headers().add<Http::Header::ContentType>("text/plain; charset='utf-8'");
  string name = request.param(":name").as<string>();
  if (request.body().empty() || name == "undefined") 
  {
    response.send(Http::Code::No_Content, "没有传入图片！");
    return;
  } // 理论上，严谨的写法，还需要判断传入的 MIME type 是不是图片

  try 
  {
    string path = "assets/images/" + name;
    std::ofstream fout(path);
    fout.write(request.body().data(), request.body().size());
    fout.close();
    
    string sqlStoreImage("INSERT INTO e_warning_map (`id`,`filename`) VALUES (1,?) ON DUPLICATE KEY UPDATE `id`=values(id),`filename`=values(`filename`)");
    auto conn = Config::pConnPool->getConnection();
    auto prp = conn.prepareStatement(sqlStoreImage.c_str());
    conn.beginTransaction();
    prp.bind(1,name);
    prp.execute();

    conn.commit();//判断执行成功后再提交事务
  }
  catch (sql_exception& e) 
  {
    string reason = "图片 " + name + " 写入数据库错误：" + e.what();
    logErr(reason);
    response.send(Http::Code::Expectation_Failed, reason);
    //prp.rollback();
    return;
  }
  catch (std::exception &e) 
  {
    string reason = "图片 " + name + " 写入目录错误：" + e.what();
    logErr(reason);
    response.send(Http::Code::Expectation_Failed, reason);
    return;
  }
  response.send(Http::Code::Ok, "图片 " + name + " 上传成功！");
}
```

## 24

#### 开发上传预警资料接口

```cpp
//commands.hpp
void uploadWarningDocument(const Rest::Request &request, Http::ResponseWriter response);

//commands.cpp
void uploadWarningDocument(const Rest::Request &request, Http::ResponseWriter response)
{
  string name = request.param(":name").as<string>();
  if (request.body().empty() || name == "undefined")
  {
    response.send(Http::Code::No_Content, "没有传入pdf文件！");
    return;
  }
	string filename = request.param(":name").as<string>();
  fs::path filePath("assets/images/");
  filePath /= filename;
	//cout<<"warning document name=	"<<filePath;
  if(!fs::exists(filePath.parent_path()))
  {
    cout << "Invalid path" << endl;
    response.send(Http::Code::Expectation_Failed, "Invalid path");
    return;
  }
  std::ofstream ofs(filePath);
  ofs.write(request.body().data(), request.body().size());
  ofs.close();    
  cout << filename << " uploaded!" << endl;
  response.send(Http::Code::Ok, filename + " uploaded!");
  catch (std::exception &e) 
  {
    cout << "Failed to upload" << endl;
    logErr(e.what());
    response.send(Http::Code::Expectation_Failed, "fail to upload " + filename);
    return;
  }
}

//router.cpp
Routes::Post(RoutesManager::router, "/upload_warningDoc/:name",
               Routes::bind(&cmd::uploadWarningDocument));
```


## 25

* 开发接口

  ```cpp
    Routes::Get(RoutesManager::router, "/devices",
                Routes::bind(&cmd::getDevices));
  ```

#### 调整预警逻辑

msg是运维组，alarm是用户组。前端页面加了个预警的功能。有预警来的时候先发给msg运维，运维去页面上点击【查询处理结果及意见】按钮，对预警进行处理或删除。后端只发给用户：蓝色的（不论是否处理）都发送，非蓝色的不发送，处理后的非蓝色发送。

## 26

#### 测试预警，功能实现

```cpp
//update_db.sql
CREATE TABLE IF NOT EXISTS e_warning_document(
`doc_name` VARCHAR(100) DEFAULT NULL COMMENT "资料名",
`upload_file_name` VARCHAR(100) DEFAULT NULL COMMENT "资料对应上传pdf文件名"
)ENGINE = InnoDB AUTO_INCREMENT=1 CHARACTER SET = utf8;
```

* 开发接口：

  ```cpp
  response.headers().add<Http::Header::ContentType>("application/json");
    queryDb(conn, sql.c_str())
        .map([&](ResultSet rslt) {
          string vals("{");
          for (int i = 0; i < clmns.size(); i++) {
            vals += "\"" + clmns[i] + "\":\"";
            vals += rslt.getString(clmns[i].c_str());
  
            vals += string("\",");
          }
          if (!vals.empty()) {
            vals[vals.size() - 1] = '}';
          } else {
            vals += "}";
          }
          response.send(Http::Code::Ok, vals.c_str());
  ```

## 27

* 修改接口`devices`的bug

#### 编写接口`save_devices`并测试

## 28

* 修改接口`save_devices`的bug，并测试通过。

  ```cpp
  //Config.hpp
  static tl::expected<bool, string> saveToJsonFile(string const &jsonPath, Document const &doc) 
  {
    FILE* fp = fopen(jsonPath.c_str(), "wb");
    if(!fp)  
      return tl::make_unexpected("failed opening file: " + jsonPath);
    
    char writeBuf[1024];
    FileWriteStream fws(fp, writeBuf, sizeof(writeBuf));
    PrettyWriter<FileWriteStream> writer(fws);
    doc.Accept(writer);
    fclose(fp);
    return true;
  }
  
  //commands.cpp
  void saveDevices(const Rest::Request &request, Http::ResponseWriter response)
  {
    response.headers().add<Http::Header::ContentType>("text/plain");
    if (request.body().empty())
    {
      response.send(Http::Code::No_Content, "The request body is empty");
      return;
    }
    if (!request.hasParam(":jsonName"))
    {
      response.send(Http::Code::Bad_Request, "缺少参数:配置文件名");
      return;
    }
    string jsonName = "etc/";
    jsonName += request.param(":jsonName").as<string>();
    
    try
    {
      // 解析请求中的json字符串
      string devicesJson = request.body().data();
      Document docReq;
      docReq.Parse(devicesJson.c_str(), devicesJson.size());
      if (!docReq.HasMember("devices"))
      {
        response.send(Http::Code::Bad_Request, "没有devices字段");
        return;
      }
  
      const Value &devices = docReq["devices"];
      if (!devices.IsArray())
      {
        response.send(Http::Code::Bad_Request, "格式不对");
        return;
      }
      if(devices.Size()<=0 || devices.Size()!=hzi::config.devicesMap.size())
      {
        response.send(Http::Code::Bad_Request, "设备数量不对");
        return;
      }
  
      // 修改内存中变量，并暂存被修改的device到devicesUpdate
      vector<Device> devicesUpdate;
      int id = 0;
      string ip = "";
      int port = 0;
      int serverPort = 0;
      for (SizeType i = 0; i < devices.Size(); i++)
      {
        const Value &device = devices[i];
        if (!device.IsObject())
          continue;
        id = device["id"].GetInt();
        ip = device["device_ip"].GetString();
        port = device["device_port"].GetInt();
        serverPort = device["server_port"].GetInt();
        Device deviceTmp{id, ip, port, serverPort};
        hzi::config.devicesMap[id] = deviceTmp;
        devicesUpdate.push_back(deviceTmp);
      }
  
      // 保存到配置文件
      auto retReadJson = Config::jsonDocFromFile(jsonName);
      if (!retReadJson.has_value())
      {
        response.send(Http::Code::Expectation_Failed, "读取配置文件失败: " + retReadJson.error());
        return;
      }
      Document &docSave = retReadJson.value();
      if(!docSave.HasMember("devices") || !docSave["devices"].IsArray())
      {
        response.send(Http::Code::Expectation_Failed, "配置文件没有devices字段: " + retReadJson.error());
        return;
      }
  
      for(int i=0;i<devicesUpdate.size();i++)
      {
        Value& device = docSave["devices"][i];
        if(!device.IsObject())
          continue;
        device["id"] = Value().SetInt(devicesUpdate[i].id);
        device["device_ip"] = Value(devicesUpdate[i].devIp.c_str(), devicesUpdate[i].devIp.size(), docSave.GetAllocator()).Move();
        device["device_port"] = Value().SetInt(devicesUpdate[i].devPort);
        device["server_port"] = Value().SetInt(devicesUpdate[i].serverPort);
      }
  
      auto retWriteJson = Config::saveToJsonFile(jsonName, docSave); // 保存到json文件
  
      if (!retWriteJson.has_value())
      {
        response.send(Http::Code::Expectation_Failed, "保存到配置文件失败: " + retWriteJson.error());
        return;
      }
      else if (retWriteJson.value() == true)
        response.send(Http::Code::Ok, "更新配置文件OK");
      return;
    }
    catch (const std::exception &e)
    {
      logErr(e.what());
      response.send(Http::Code::Expectation_Failed, e.what());
    }
  }
  ```

#### 修改接口`save_devices`功能，device_ip和device_port不能修改。根据id修改server_port。并改名称为`updateServerPort`

```cpp
void updateServerPort(const Rest::Request &request, Http::ResponseWriter response)
{
  response.headers().add<Http::Header::ContentType>("text/plain");
  if (request.body().empty())
  {
    response.send(Http::Code::No_Content, "The request body is empty");
    return;
  }
  if (!request.hasParam(":jsonName"))
  {
    response.send(Http::Code::Bad_Request, "缺少参数:配置文件名");
    return;
  }
  string jsonName = "etc/";
  jsonName += request.param(":jsonName").as<string>();
  
  try
  {
    // 解析请求中的json字符串
    string devicesJson = request.body().data();
    Document docReq;
    docReq.Parse(devicesJson.c_str(), devicesJson.size());
    if (!docReq.HasMember("devices"))
    {
      response.send(Http::Code::Bad_Request, "没有devices字段");
      return;
    }

    const Value &devices = docReq["devices"];
    if (!devices.IsArray())
    {
      response.send(Http::Code::Bad_Request, "格式不对");
      return;
    }
    if(devices.Size()<=0 || devices.Size()!=hzi::config.devicesMap.size())
    {
      response.send(Http::Code::Bad_Request, "基站数量不对");
      return;
    }

    // 修改内存中变量，并暂存被修改的device到devicesUpdate
    map<int,int> devicesPortUpdate;
    int id = 0;
    string ip = "";
    int port = 0;
    int serverPort = 0;
    for (SizeType i = 0; i < devices.Size(); i++)
    {
      const Value &device = devices[i];
      if (!device.IsObject())
        continue;
      /*
      id = device["id"].GetInt();
      ip = device["device_ip"].GetString();
      port = device["device_port"].GetInt();
      serverPort = device["server_port"].GetInt();
      Device deviceTmp{id, ip, port, serverPort};
      hzi::config.devicesMap[id] = deviceTmp;
      */
      id = device["id"].GetInt();
      serverPort = device["server_port"].GetInt();
      hzi::config.devicesMap[id].serverPort = serverPort;
      devicesPortUpdate.insert_or_assign(id,serverPort);
    }

    // 保存到配置文件
    auto retReadJson = Config::jsonDocFromFile(jsonName);
    if (!retReadJson.has_value())
    {
      response.send(Http::Code::Expectation_Failed, "读取配置文件失败: " + retReadJson.error());
      return;
    }
    Document &docSave = retReadJson.value();
    if(!docSave.HasMember("devices") || !docSave["devices"].IsArray())
    {
      response.send(Http::Code::Expectation_Failed, "配置文件没有devices字段: " + retReadJson.error());
      return;
    }

    for(int i=0;i<docSave["devices"].Size();i++)
    {
      Value& device = docSave["devices"][i];
      if(!device.IsObject())
        continue;
      int id;
      if(device.HasMember("id"))
        id = device["id"].GetInt();
      else
        continue;
      if(devicesPortUpdate.find(id)==devicesPortUpdate.end())
        continue;

      //device["id"] = Value().SetInt(devicesUpdate[i].id);
      //device["device_ip"] = Value(devicesUpdate[i].devIp.c_str(), devicesUpdate[i].devIp.size(), docSave.GetAllocator()).Move();
      //device["device_port"] = Value().SetInt(devicesUpdate[i].devPort);
      if(!device.HasMember("server_port") || !device["server_port"].IsInt())
        continue;
      //device["server_port"] = Value().SetInt(devicesPortUpdate[id]);
      device["server_port"].SetInt(devicesPortUpdate[id]).Move();
    }

    auto retWriteJson = Config::saveToJsonFile(jsonName, docSave); // 保存到json文件

    if (!retWriteJson.has_value())
    {
      response.send(Http::Code::Expectation_Failed, "保存到配置文件失败: " + retWriteJson.error());
      return;
    }
    else if (retWriteJson.value() == true)
      response.send(Http::Code::Ok, "更新配置文件OK");
    return;
  }
  catch (const std::exception &e)
  {
    logErr(e.what());
    response.send(Http::Code::Expectation_Failed, e.what());
  }
}
```


## 31

#### 看数据采集代码

#### 不知道为什么，单步到`http_server.cpp/start_http_server`报错

# 2025/4

## 1

* 看接收原始帧代码`receiver.cpp`
* 看`sender.cpp`
* 笔记整理数据库表格说明

## 2

* 看接收原始帧处理`recv_data.cpp`

* 接口`auth_user`

## 3

* session/cookie/MIME概念

## 4

* bug：提取信号，通道2无数据

## 7

* 范各庄的提取图像通道2无波形
* 和刘金锁沟通：相关原理

## 8

* **TODO：开会沟通范各庄矿的2通道无波形问题，以及其他问题**
* 需求分析：距离迎头通道号不是1的时候，计算通道坐标问题修改
* **TODO：搭建测试环境**

## 9

#### fix_bug：修改基准通道，返回对应坐标。

> `迎头-偏移距`得到`基准通道坐标`。先前是1，某次挪动后改为3，那么新的基准就是3。
>
> `挪动后迎头-挪动后偏移距===>新基准通道3坐标`
>
> 挪动后的3的坐标-挪动前3的坐标（注意不是减挪动前1的坐标），即为delta各个通道坐标都要增加的值。

```cpp
// //从之前写死取chn_no=1的检波器作为距离迎头最近检波器，改为1~16号选择某个检波器作为最近
    // //但由于该代码发生在系统初始化阶段，是否会影响初始化后续使用hzi::firstChn_locx的过程，未知。
    // auto rslt2 = conn.executeQuery("SELECT loc_x FROM `e_chns_config` WHERE type_id = 0 AND chn_no=?", 
    //                                 hzi::firstChn_no);
    // if (rslt2.next()) 
    // {
    //   // 获取第一个检波器坐标
    //   hzi::firstChn_locx = rslt2.getDouble("loc_x");
    // }
```

修改：

> * 【第一个通道的坐标】获取发生在系统初始化阶段（Config.hpp-779）
>
> * 可能用到【第一个通道的坐标】的地方（client_handle_data.cpp-420）

可能影响：

> 未知e

* 发现问题：为什么Config.hpp-1352的init中增加日志不打印。

  要编译为debug才会打印，release不打印。

## 10

* 提交【修改基准通道】（是否会造成其他bug未知，待观察）

* gyx：预警表e_warning_info有记录，但前端处理页面不显示。发现是`commands.cpp-1733的isNull`查询接口，只要有字段为null，查出结果就会返回`**204 No Content**：请求成功，但响应体为空`。

  经查，发现表e_warning_info的suggestion字段在建表时，写的是`DEFAULT NULL`。

  `update_db.sql`新增修改该字段为`NOT NULL DEFAULT "请在此处写处理意见"`

## 14

#### fix：登陆页面若密码带有@则无法通过验证的情况

> * url中的@被编码为%40传送到服务器的，需要解码。调用`unescape`，并在`escapedChars`中增加了`{"%40", "@"}`的映射关系。
>
> * `verifyUser`函数中的`queryDb`函数通过占位符的方式，查不出字段带有@的记录，即使表里有对应记录。
>
>   ```cpp
>   bool verifyUser(string user, string pswd) {
>   auto conn = hzi::config.pConnPool->getConnection();
>   // bool ret = cmd::queryDb(conn,
>   //                         "SELECT * FROM e_users where user_name=? and "
>   //                         "password=PASSWORD(?)",
>   //                         user, pswd) ? true : false;
>                             
>   /*
>     很奇怪：ret1=true,ret2=false,ret=true
>     queryDb()通过占位符的方式传入的参数中若包含@字符，查不出来。
>     所以导致带有@的密码无法通过验证。
>     而通过拼字符串的方式，直接调用executeQuery()则可以查到。
>   */
>   // bool ret1 = cmd::queryDb(conn,
>   //                         "SELECT * FROM e_users where user_name='jinchuanweizhen' and "
>   //                         "password=PASSWORD('JckyDiceke115113.@')") ? true : false;
>   // std::cout<<"ret1="<<ret1<<std::endl;
>                         
>     ```cpp
>                       
>   // bool ret2 = cmd::queryDb(conn,
>   //                         "SELECT * FROM e_users where user_name=? and "
>   //                         "password=PASSWORD(?)", user, pswd) ? true : false;
>     // std::cout<<"ret2="<<ret2<<std::endl;
>                         
>     string sqlVerifyUser = "SELECT * FROM e_users where `user_name`='" + user + "' and `password`=PASSWORD('" + pswd + "')";
>   auto ret = conn.executeQuery(sqlVerifyUser.c_str());
>   if(ret.next())
>   {
>     return true;
>   }
>   return false;
>   // return ret;
>   }

#### 接口`save_devices`回调函数修改：从只修改server_port改到修改devip/devport/server_port。

若请求修改的id在json中不存在，不做任何操作（不添加）；存在，则修改。

修改接口名为`update_devices`

## 15

#### 拷贝了所需表到本地以及原始3/6数据到路径下，但系统手动历史计算时，并没有生成9类型文件也没有出提取图。找问题所在。

## 16

* 偏移距问题的修改，升级到【新集二矿随掘系统】进行测试。fix小bug，通道号字段名在json中的前后端约定不一致，从`chnNO`改为`chn_no`。

  ![image-20250416091925883](D:\notes\笔记Img\image-20250416091925883.png)

## 17

* 查找bug：

  ![image-20250417100530544](D:\notes\笔记Img\image-20250417100530544.png)

* bug：

  ![image-20250417131756797](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20250417131756798)

## 18

#### fix bug：接口`firstChnNO`，偏移距基准通道页面打开不显示以及发送chnNO发送null时崩掉的问题。

现在打开页面时首先查询当前使用的基准通道号。

* 排查园区实验系统，树形图不显示。

* 排查范各庄，全时背景切换问题。

## 21

* bug

  ![image-20250421090516183](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20250421090516182)

  ![image-20250421090445316](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-202504210904453161)

  ![image-20250421090558922](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20250421090558922666)

#### fix bug：系统设置改为全时，但实时数据显示的提取图还是背景

* 点击一次【同步】，触发接口`/config/e_mining_PCOSignalParm`，修改了表`e_mining_PCOSignalParm_history`和表`e_mining_PCOSignalParm`。

  前端的【滤波】下增加了【拼接数据类型】，可设定历史计算使用的数据类型是3或6，也即修改表`e_mining_PCOSignalParm_history`。

  但前者接口json传了class_id，后者没传。

  又因为表e_mining_PCOSignalParm的class_id字段默认值为3。未传值导致自动变更为3。
  
  并且，`hzi::mining_classId`初始值为3，实时计算时函数`threadHanleData`中`mergeMs`拼帧前，没有从实时计算参数表`e_mining_PCOSignalParm`中查询class_id，而是直接用`hzi::mining_classId`。而历史计算的接口函数`handerSignalProcess`中，`mergeMs`拼帧前使用`hzi::mining_classId`查的却是实时表`e_mining_PCOSignalParm`。

![image-20250421093139786](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-202504210931397861)

![image-20250421093209715](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20250421093209715.png)

![image-20250421095117706](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20250421095117706.png)

![image-20250421095134819](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20250421095134819.png)

![image-20250421100705134](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-202504211007051341)

![image-20250421102226600](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-202504211022266001)

* 会议：现有问题，以及搭建测试环境

## 22

#### fix bug：实时计算类型写死为背景

* 原本实时计算的类型不根据实时表`e_mining_PCOSignalParm1`中的`class_id`来，而是直接用的` hzi::mining_classId`的初始值3。见`ms_mining.cpp-5538`。改为：先从表里查`class_id`。
* 历史计算中接口`/handerSignal/:from_time/:to_time/:time_len/:devId`和`/handerSignalMining/:from_time/:to_time/:time_len/:devId`使用的回调函数`handerSignalProcess`和`handerSignalProcess_mining`，中在使用`hzi::mining_classId`前都是查的实时表`e_mining_PCOSignalParm`。应改为查历史表`e_mining_PCOSignalParm_history`。
* 并让前端在页面【随掘地震】中设定的参数入库到历史表`e_mining_PCOSignalParm_history`，而【系统-随掘监测-实施参数】中设定的参数入库到实时表`e_mining_PCOSignalParm`。

#### bug：~~潘集三矿提取图不对~~。

历史重算得到的图正确，但实时的不对。考虑是实时写死是拿背景数据计算的bug，历史重算的图拿全时算的。

![image-20250422154644551](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-202504221546445512)

#### TODO：写预警重算/~~下载接口~~

![image-20250422155135640](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-202504221551356401)

## 23

#### 写接口：文件上传，下载。暂定支持格式：png/jpg/jpeg/gif/pdf/csv/xlsx

将原接口`/download_WarningDoc/:mime/:name`中`Warning`去掉，作为通用文件上传/下载接口。

```cpp
/upload_Doc/:mime/:name
/download_Doc/:mime/:name
```

## 24

#### 写接口：预警重算。

拆分getWarningMsg()函数为2个函数，一个获取时间段，一个对时间段内的e_msevt_rslts的记录进行查询。

#### 整理文档

## 25

#### 写功能：增加回采位置历史记录

#### 预警重算接口增加异常处理

## 27

#### 排查代码：是否是，不论勾不勾选道间均衡，再提取过程都做了道间均衡？

## 28

会议：手头还剩的事情。节点的进度，硬件完成了吗。

#### 预警电法和地震分开，并测试

## 29

