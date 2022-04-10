---
title: unityMqtt
date: 2022-04-10 21:46:35
tags:
---
## 思路

采用Unity3d MQTT插件搭建mqtt的客户端

<!-- more -->

## 流程

1. 克隆[Unity3d MQTT插件](https://github.com/vovacooper/Unity3d_MQTT)库
2. 将unitypackage导入项目
3. 编写逻辑

## 代码

```Csharp
// 纯逻辑
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using TMPro;
using UnityEngine;
using UnityEngine.UI;
using uPLibrary.Networking.M2Mqtt;
using uPLibrary.Networking.M2Mqtt.Messages;

public class GetMqttMes : MonoBehaviour
{

    public TMP_InputField UrlInput;
    public TMP_InputField PortInput;
    public TMP_InputField UserNameInput;
    public TMP_InputField Password;
    public TMP_InputField Topic;
    public TMP_InputField Message;
    public TMP_Text SublistText;
    public TMP_Text ReciveText;
    
    public Button UpdateButton;
    public Button SubscribeButton;
    public Button AddButton;
    public Button ClearButton;
    public Button ConnectButton;
    public Button DisConnectButton;

    public SpriteRenderer SpriteRender;
    private MqttClient client;
    private List<string> SubList;

    private string ReciveString;
    
    // Start is called before the first frame update
    void Start()
    {
        UpdateButton.onClick.AddListener(UpdateMqttMess);
        SubscribeButton.onClick.AddListener(SubscribeMqtt);
        AddButton.onClick.AddListener(AddList);
        ClearButton.onClick.AddListener(Clear);
        ConnectButton.onClick.AddListener(Connect);
        DisConnectButton.onClick.AddListener(DisConnect);

        DisConnectButton.enabled = false;
        ClearButton.enabled = false;
        AddButton.enabled = false;
        SubscribeButton.enabled = false;
        UpdateButton.enabled = false;

        SubList = new List<string>();
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    void UpdateMqttMess()
    {
        if (client == null || string.IsNullOrEmpty(Topic.text) || string.IsNullOrEmpty(Message.text))
        {
            return;
        }
        
        client.Publish(Topic.text, System.Text.Encoding.UTF8.GetBytes(Message.text), MqttMsgBase.QOS_LEVEL_EXACTLY_ONCE, true);
    }

    void SubscribeMqtt()
    {
        if (UrlInput.text == "" || UserNameInput.text == "" || Password.text == "")
        {
            return;
        }
        Debug.Log(SubList);
        client.Subscribe(SubList.ToArray(), new byte[] { MqttMsgBase.QOS_LEVEL_EXACTLY_ONCE } );
    }

    void AddList()
    {
        if (Topic.text == "")
        {
            return;
        }
        
        SubList.Add(Topic.text);
        if (string.IsNullOrEmpty(SublistText.text)) SublistText.text = Topic.text;
        else SublistText.text = SublistText.text + "," +  Topic.text;
        Topic.text = "";
    }

    void Clear()
    {
        SubList.Clear();
        SublistText.text = "";
    }

    void Connect()
    {
        if (UrlInput.text == "" || UserNameInput.text == "" || Password.text == "" || PortInput.text == "")
        {
            return;
        }

        string clientId = Guid.NewGuid().ToString();

        try
        {
            client = new MqttClient(IPAddress.Parse(UrlInput.text),int.Parse(PortInput.text),false,null);
            client.MqttMsgPublishReceived += MessReceived;
            client.Connect(clientId, UserNameInput.text, Password.text);
        }
        catch (Exception e)
        {
            Debug.Log(e.Message);
        }

        ConnectButton.enabled = false;
        DisConnectButton.enabled = true;
        ClearButton.enabled = true;
        AddButton.enabled = true;
        SubscribeButton.enabled = true;
        UpdateButton.enabled = true;
        SpriteRender.color = Color.green;
    }

    void DisConnect()
    {
        if(client == null) return;
        
        client.Disconnect();
        
        DisConnectButton.enabled = false;
        ConnectButton.enabled = true;
        ClearButton.enabled = false;
        AddButton.enabled = false;
        SubscribeButton.enabled = false;
        UpdateButton.enabled = false;
        SpriteRender.color = Color.red;
    }

    void MessReceived(object sender,MqttMsgPublishEventArgs e)
    {
        string text = "[" + e.Topic + "]:" + e.Message;
        ReciveString = text;
        Debug.Log(e.Message);
    }
}
```