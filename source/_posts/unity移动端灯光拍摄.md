---
title: unity移动端灯光拍摄
date: 2022-04-10 21:46:35
tags:
---
## 思路

调用设备摄像头接口，获取设备图像，将图像上传至后端，调用C++程序处理生成json文件

<!-- more -->

## 代码

```Csharp
using System.Collections;
using System;
using System.Collections.Generic;
using UnityEngine.UI;
using UnityEngine;
using UnityEngine.Networking;

public class WebCam : MonoBehaviour
{
    // Start is called before the first frame update
    WebCamTexture WebTexture;
    public Image CamImage;
    public MeshRenderer meshRender;
    public Button StartUpload;
    public InputField UrlInput;
    public string JsonUrl = "/GetPosForm";
    public string FileUrl = "/GetFileForm";
    private byte[] PNGImage;
    void Start()
    {
        // 开启协程 
        StartCoroutine(OpenCamera());
        StartUpload.onClick.AddListener(Upload);
        UrlInput.text = "http://192.168.50.148:3000";
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    IEnumerator OpenCamera()
    {
        // 等待用户许可访问摄像机
        yield return Application.RequestUserAuthorization(UserAuthorization.WebCam);
        if (Application.HasUserAuthorization(UserAuthorization.WebCam))
        {
            WebCamDevice[] devices = WebCamTexture.devices; // 获取设备
            string deviceName = devices[0].name;
            
            // 将设备图像绑定至WebTexure
            WebTexture = new WebCamTexture(deviceName,1920,1080,60);
            // 指定网格渲染图像为 WebTexure
            meshRender.material.mainTexture = WebTexture;
            WebTexture.Play();
        }
    }

    IEnumerator SaveImage(WebCamTexture Image)
    {
        // 根据传入贴图的大小与色彩颜色创建Textrue2D
        Texture2D Texture2DPic = new Texture2D(Image.width, Image.height, TextureFormat.ARGB32, true);
        
        // 保存当前图像
        Texture2DPic.SetPixels(Image.GetPixels());
        Texture2DPic.Apply();
        
        // 转码至PNG
        PNGImage = Texture2DPic.EncodeToPNG();
        yield return StartCoroutine(UploadFile());
    }

    public void Upload()
    {
        StartCoroutine(SaveImage(WebTexture));
        // Debug.Log(GetTimeOfDay());
    }
    
    IEnumerator UploadJson()
    {
        Debug.Log("Begin upload Json ...");
        WWWForm form = new WWWForm();
        form.AddField("Test",123);
        string Url = UrlInput.text + JsonUrl;
        using (UnityWebRequest www = UnityWebRequest.Post(Url, form))
        {
            yield return www.SendWebRequest();

            if (www.result == UnityWebRequest.Result.ConnectionError)
            {
                Debug.Log(www.error);
            }
            else
            {
                string text = www.downloadHandler.text;
                Debug.Log("服务器返回值" + text);
            }
        }
        
    }
    
    IEnumerator UploadFile()
    {
        Debug.Log("Being upload file");
        WWWForm form = new WWWForm();
        
        form.AddBinaryData("file",PNGImage,GetTimeOfDay() + ".PNG");
        
        string Url = UrlInput.text + FileUrl;
        Debug.Log(Url);
        using (UnityWebRequest www = UnityWebRequest.Post(Url, form))
        {
            yield return www.SendWebRequest();

            if (www.result == UnityWebRequest.Result.ConnectionError)
            {
                Debug.Log(www.error);
            }
            else
            {
                string text = www.downloadHandler.text;
                Debug.Log("服务器返回值" + text);
            }
        }
    }

    public string GetTimeOfDay()
    {
        //获取当前时间
        var hour = DateTime.Now.Hour;
        var minute = DateTime.Now.Minute;
        var second = DateTime.Now.Second;
        var year = DateTime.Now.Year;
        var month = DateTime.Now.Month;
        var day = DateTime.Now.Day;
        
        return string.Format("{0:D2}:{1:D2}:{2:D2}_" + "{3:D4}_{4:D2}_{5:D2}", hour, minute, second, year, month, day);
    }
}

```