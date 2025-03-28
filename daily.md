# 2025/3/23

* 开发上传预警底图资料

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

# 2025/3/24

* 开发上传预警资料接口

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

  

# 2025/3/25

* 开发接口

  ```cpp
    Routes::Get(RoutesManager::router, "/devices",
                Routes::bind(&cmd::getDevices));
  ```

* 调整预警逻辑

  msg是运维组，alarm是用户组。前端页面加了个预警的功能。有预警来的时候先发给msg运维，运维去页面上点击【查询处理结果及意见】按钮，对预警进行处理或删除。后端只发给用户：蓝色的（不论是否处理）都发送，非蓝色的不发送，处理后的非蓝色发送。

# 2025/3/26

* 测试预警，功能实现

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


# 2025/3/27

* 修改接口`devices`的bug
* 编写接口`save_devices`并测试

# 2025/3/28

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

* 修改接口`save_devices`功能，device_ip和device_port不能修改。根据id修改server_port。并改名称为`updateServerPort`

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

  
