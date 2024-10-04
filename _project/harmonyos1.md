---
title: "初试HarmonyOS - 实现一个简单的APP"
collection: project
type: "Project"
permalink: /project/harmonyos-1
date: 2024-10-04
---

摘要：基于HarmonyOS的ArkTS框架，我实现了一个简单的APP进行数据获取和展示

# 环境介绍
&emsp;&emsp;HarmonyOS是华为开发的操作系统，目前已经更新到了HarmonyOS NEXT——也就是常说的纯血鸿蒙，有广阔的发展前景。

&emsp;&emsp;我们团队中的项目需要开发一个移动端的APP进行数据展示，本着追逐风口和支持国产的想法，我们选择基于HarmonyOS进行APP的开发。

&emsp;&emsp;这是我第一次进行APP开发，当时的HarmonyOS还是支持JAVA和ArkTS两种语言进行开发，因为官方在ArkTS方面有相关说明，所以我选用ArkTS语言进行开发，而基于ArkTS的声明式UI也是比较简单，容易上手。

&emsp;&emsp;我们使用华为的DevEco Studio作为IDE，选用的SDK版本为3.1.0(API 9)

# 产品说明
&emsp;&emsp;我们的项目是对轴承进行故障检测和寿命预测，并将检测和预测的结果存储在服务器中，由移动端进行数据的获取和展示。因此，APP的主要需求就是从网络中获取数据和恰当的数据展示。

&emsp;&emsp;在具体的实现中，我使用6个Page和一个常量类实现主要功能。

&emsp;&emsp;* Constant.ets - 存放一些常量，例如一些文字大小，颜色等等
&emsp;&emsp;* Index.ets - 初始化界面，进行一些数据的加载
&emsp;&emsp;* Login.ets - 登陆界面，进行用户登陆和身份验证操作
&emsp;&emsp;* Register.ets - 注册界面，首次使用用户注册
&emsp;&emsp;* Load.ets - 加载界面，用于轴承数据的加载
&emsp;&emsp;* Table.ets - 主页，对轴承数据进行归档展示
&emsp;&emsp;* Detail.ets - 详情界面，对每个轴承的详细信息和检测结果进行展示

# 代码介绍

## Constant
```typescript
export default class Constant{
  //登陆标题大小
  static readonly LOGIN_SIZE:number=30
  //登陆标题颜色
  static readonly LOGIN_COLOR:string='#0387FE'
  //输入框长度
  static readonly INPUT_WIDTH:number=350
  //输入框高度
  static readonly INPUT_HEIGHT:number=60
  //按钮宽度
  static readonly BUTTON_WIDTH:number=150
  //按钮高度
  static readonly BUTTON_HEIGHT:number=50
  //登录按钮颜色
  static readonly BUTTON_COLOR:string='#0171FF'
  //列表单元尺度
  static readonly SPACE_SIZE:number=15
  //名称
  static readonly APP_NAME:string='Argbearing'

  //API
  static readonly API:string= 'http://172.20.10.3/fetch_faultinfo.php'
  static readonly API1:string='http://172.20.10.3/select_number.php'
  static readonly API2:string='http://172.20.10.3/select_machinebearinginfo_2.php'
  static readonly API3:string='http://172.20.10.3/change_pwd.php'
  static readonly API4:string='http://172.20.10.3/register.php'
  static readonly API5:string='http://172.20.10.3/search_pwd.php'
  static readonly API6:string='http://172.20.10.3/select_machinebearingrul_2.php'
  static readonly API7:string='http://172.20.10.3/anomaly_det.php'
  static readonly API8:string='http://172.20.10.3/rul_predict.php'

  //welcolor
  static readonly wel_col_0:string ='#4fabfe'
  static readonly wel_col_1:string ='#0166e6'

  //loadcoler
  static readonly loa_col_0:string ='#ffffff'
  static readonly loa_col_1:string ='#cccccc'
}
```

## Index
```typescript
import router from '@ohos.router';
import Constant from '../common/Constant'

@Entry
@Component
struct Index {

  @State opacityValue:number=0.3;

  onPageShow(){
    this.opacityValue=1;
  }

  build(){
    Column(){
      Column(){
        Text('ArgBearing').fontColor(Color.White).fontSize(50).fontWeight(FontWeight.Bold).margin({top:150}).fontStyle(FontStyle.Italic)
      }
      .width("100%").height("50%")
      .margin({top:350})
      .alignItems(HorizontalAlign.Center).opacity(this.opacityValue)
      Text('Powered by 海纳百川')
        .margin({top:150})
        .textAlign(TextAlign.Center)
        .fontColor(Color.White).fontSize(20).fontWeight(FontWeight.Bold)
    }.width('100%').height('100%')
    .linearGradient({
      colors:[
        [Constant.wel_col_0,0],[Constant.wel_col_1,1]
      ]
    })
    .animation({
      duration:1500,
      iterations:1,
      onFinish:()=>{
        console.info('go to login')
        console.info(' ')
        router.replaceUrl({
          //url: 'pages/Login',
          url: 'pages/load',
          params: {
             f:0,
             mn:-1,
             bn:-1,
            // ipAddress:"172.20.10.3"
          }ets
        }, router.RouterMode.Single)
      }
    })
  }
}
```
&emsp;&emsp;因为我们不会再回到初始界面，所以我们选用router.replaceUrl进行跳转

## Login
```typescript
import Constant from '../common/Constant'
import router from '@ohos.router'
import http from '@ohos.net.http';

function openDialog(text: string) {
  AlertDialog.show(
    {
      //title: 'title',
      message: text,
      autoCancel: true,
      alignment: DialogAlignment.Bottom,
      offset: { dx: 0, dy: -20 },
    }
  )
}

@Entry
@Component
struct login{

  @State logo : string = '天    穹'
  @State username : string = ''
  @State password : string = ''
  //@State ipAddress : string = ''

  build(){
    Row(){
      Column(){
        Image($r('app.media.APP_logo')).width(150)

        Text('ArgBearing').fontColor(Constant.LOGIN_COLOR).fontSize(50).fontWeight(FontWeight.Bold).margin({top:30}).fontStyle(FontStyle.Italic)

        TextInput({placeholder:'请输入用户名'})
          .margin({top:30})
          .width(Constant.INPUT_WIDTH)
          .height(Constant.INPUT_HEIGHT)
          .onChange((value:string)=>{
            this.username=value;
          })

        TextInput({placeholder:'请输入用户密码'}).type(InputType.Password)
          .margin({top:30})
          .width(Constant.INPUT_WIDTH)
          .height(Constant.INPUT_HEIGHT)
          .onChange((value:string)=>{
            this.password=value;
          })

        Button('登录')
          .width(Constant.BUTTON_WIDTH)
          .height(Constant.BUTTON_HEIGHT)
          .margin({top:40})
          .backgroundColor(Constant.BUTTON_COLOR)
          .onClick(async ()=>{
            let httpRequest = http.createHttp();
            //let url = "http://"+this.ipAddress+Constant.API5;
            let url = Constant.API5;
            let promise = httpRequest.request(
              url,
              {
                method: http.RequestMethod.POST,
                extraData:'name='+this.username,
                connectTimeout: 60000,
                readTimeout: 60000,
                header: {
                  "Content-Type": "application/x-www-form-urlencoded"
                }
              });
            await promise.then((data) => {
              if (data.responseCode === http.ResponseCode.OK) {
                if(data.result===this.password){
                  console.log("登录成功,go to load")
                  router.replaceUrl({
                    url:'pages/load',
                    params:{
                      f:0,
                      mn:-1,
                      bn:-1,
                      //ipAddress:this.ipAddress
                    }
                  }, router.RouterMode.Single)
                }
                else{
                  openDialog("密码错误，请重试")
                }
              }
             }).catch((err) => {
              openDialog(err)
              console.info('error:' + JSON.stringify(err));
             });
          })
        Flex(){
          Text('尚未拥有账户？ 注册')
            .fontColor(Constant.LOGIN_COLOR)
            .onClick(()=>{
              router.pushUrl({
                url: 'pages/Register',
                params: {
                  //ipAddress:this.ipAddress
                }
              }, router.RouterMode.Single)
            })
        }
        .margin(30)
      }
      .width('100%')
    }
    .height('100%')
  }
}
```
&emsp;&emsp;这里我们使用httpRequest从服务器获取数据，并与输入的密码对比验证。

&emsp;&emsp;同时，我们使用AlertDialog.show作提示弹窗。

## Register
```typescript
import router from '@ohos.router'
import http from '@ohos.net.http';
import Constant from '../common/Constant'

//let ipAddress=router.getParams()['ipAddress']

@Entry
@Component
struct Register {
  @State message: string = '观星'
  @State username: string = ''
  @State password: string = ''
  @State email: string = ''

  build() {
      Column() {
        Blank().height('2%').width('100%').backgroundColor('#ffffff')
        Row(){
          Image($r('app.media.ic_public_left_filled'))
            .interpolation(ImageInterpolation.High)
            .height("60%")
            .margin({left:15})
            .backgroundColor(Color.White)
            .onClick(()=>{
              router.back();
            })

        }.width('100%')
        .backgroundColor(Color.White).height("5%")

        Image($r('app.media.APP_logo')).width(150).margin({top:200})

        Text('ArgBearing').fontColor(Constant.LOGIN_COLOR).fontSize(50).fontWeight(FontWeight.Bold).margin({top:20}).fontStyle(FontStyle.Italic)

        TextInput({ placeholder: '用户名' })
          .margin({ top: 20 })
          .width(Constant.INPUT_WIDTH)
          .height(Constant.INPUT_HEIGHT)
          .onChange((value: string) => {
            this.username = value
          })

        TextInput({ placeholder: '邮箱' })
          .margin({ top: 30 })
          .width(Constant.INPUT_WIDTH)
          .height(Constant.INPUT_HEIGHT)
          .onChange((value: string) => {
            this.email = value
          })

        TextInput({ placeholder: '用户密码' })
          .type(InputType.Password)
          .margin({ top: 30 })
          .width(Constant.INPUT_WIDTH)
          .height(Constant.INPUT_HEIGHT)
          .onChange((value: string) => {
            this.password = value
          })

        Button('注册')
          .width(Constant.BUTTON_WIDTH)
          .height(Constant.BUTTON_HEIGHT)
          .margin({ top: 40 })
          .backgroundColor(Constant.BUTTON_COLOR)
          .onClick(() => {

            let httpRequest = http.createHttp();
            //let url = "http://"+ipAddress+Constant.API4;
            let url = Constant.API4;
            let promise = httpRequest.request(
              url,
              {
                method: http.RequestMethod.POST,
                extraData:'name='+String(this.username)+'&pwd='+String(this.password),
                connectTimeout: 60000,
                readTimeout: 60000,
                header: {
                  "Content-Type": "application/x-www-form-urlencoded"
                }
              });
            promise.then((data) => {
              if (data.responseCode === http.ResponseCode.OK) {
                console.info('Result:' + data.result);
                console.info('code:' + data.responseCode);
              }
            }).catch((err) => {
              console.info('error:' + JSON.stringify(err));
            });

            router.replaceUrl({
              url: 'pages/Login',
              params: {
                username: this.username,
                password: this.password,
                email: this.email
              }
            }, router.RouterMode.Single)
          })
      }
  }
}
```
&emsp;&emsp;这里要注意，当我们传送的extraData数量为两个及以上时，需要使用“+”将其连接起来，而非官方文档中的";"(可能官方文档还没来得及更改)
```typescript
extraData:'name='+String(this.username)+'&pwd='+String(this.password)
```

## Load
```typescript
import router from '@ohos.router';
import http from '@ohos.net.http';
import Constant from '../common/Constant'

let f=router.getParams()['f']
let mn=router.getParams()['mn']
let bn=router.getParams()['bn']

export let dataList : string[][][][]=[]
export let mbNum:number[]=[]
export let pre:string[][][][]=[]

@Entry
@Preview
@Component
struct Load {

  @State opacityValue:number=0.3;

   async onPageShow(){
    this.opacityValue=1;

     console.log("arrival load")

     let httpRequest1 = http.createHttp();
     //let url= "http://"+ipAddress+Constant.API1;
     let url= Constant.API1;
     console.log("begpro")
     let promise = httpRequest1.request(
       url,
       {
         method: http.RequestMethod.GET,
         connectTimeout: 60000,
         readTimeout: 60000,
         header: {
           'Content-Type': 'application/json'
         }
       });
     console.log("begpromise")
     await promise.then((data) => {
       console.log("pro1")
       if (data.responseCode === http.ResponseCode.OK) {
         let dataP = JSON.parse(JSON.parse(JSON.stringify(data.result)))
         for(let i=0;i<dataP.length;i++){
           //mbNum的索引为机器编号-1（0～n-1），存储数值为其轴承数量
           let num = Number(dataP[i]["machineNumber"]);
           mbNum[num-1]=Number(dataP[i]["bearingNumber"])
         }
       }
     })
     console.log("promiseover")

     for (let i = 0;i < mbNum.length; i++) {
       dataList[i] = []
       //let url2 = "http://" + ipAddress + Constant.API2;
       let url2 = Constant.API2;
       let httpRequest2 = http.createHttp();
       let promise2 = httpRequest2.request(
         url2,
         {
           method: http.RequestMethod.POST,
           //传参，index+1表示机器编号
           extraData: 'machineNumber=' + (i + 1).toString(),
           connectTimeout: 60000,
           readTimeout: 60000,
           header: {
             "Content-Type": "application/x-www-form-urlencoded"
           }
         });
       console.log("begpromise2  "+i)
       await promise2.then((data) => {
         if (data.responseCode === http.ResponseCode.OK) {
           //将读到的数据转换为对象
           let dataBTP = JSON.parse(JSON.parse(JSON.stringify(data.result)))
           // if(dataBTP!=null)
           //   console.log(JSON.stringify(dataBTP))
           let k
           let bk:number[]=[]
           for (let j = 0;j < dataBTP.length; j++) {
             //轴承编号也从0开始
             let BNum = Number(dataBTP[j]["bearingNumber"]) - 1
             //console.log(BNum.toString())
             if (dataList[i][BNum] == null) {
               //若为空则初始化
               dataList[i][BNum] = []
               bk[BNum]=0
               k = 0
             }
             dataList[i][BNum][bk[BNum]] = []
             dataList[i][BNum][bk[BNum]][0] = JSON.parse(JSON.stringify(dataBTP[j]["detectSN"]))
             dataList[i][BNum][bk[BNum]][1] = JSON.parse(JSON.stringify(dataBTP[j]["machineNumber"]))
             dataList[i][BNum][bk[BNum]][2] = JSON.parse(JSON.stringify(dataBTP[j]["bearingNumber"]))
             dataList[i][BNum][bk[BNum]][3] = JSON.parse(JSON.stringify(dataBTP[j]["faultDia1"]))
             dataList[i][BNum][bk[BNum]][4] = JSON.parse(JSON.stringify(dataBTP[j]["faultLoc1"]))
             dataList[i][BNum][bk[BNum]][5] = JSON.parse(JSON.stringify(dataBTP[j]["faultScore1"]))
             dataList[i][BNum][bk[BNum]][6] = JSON.parse(JSON.stringify(dataBTP[j]["faultDia2"]))
             dataList[i][BNum][bk[BNum]][7] = JSON.parse(JSON.stringify(dataBTP[j]["faultLoc2"]))
             dataList[i][BNum][bk[BNum]][8] = JSON.parse(JSON.stringify(dataBTP[j]["faultScore2"]))
             dataList[i][BNum][bk[BNum]][9] = JSON.parse(JSON.stringify(dataBTP[j]["faultDia3"]))
             dataList[i][BNum][bk[BNum]][10] = JSON.parse(JSON.stringify(dataBTP[j]["faultLoc3"]))
             dataList[i][BNum][bk[BNum]][11] = JSON.parse(JSON.stringify(dataBTP[j]["faultScore3"]))
             dataList[i][BNum][bk[BNum]][12] = JSON.parse(JSON.stringify(dataBTP[j]["detectTime"]))
             bk[BNum]++
           }
         }
       })
       console.log("promise2  "+i+"over")
     }

     for (let i = 0;i < mbNum.length; i++) {
       pre[i] = []
       //let url3 = "http://" + ipAddress + Constant.API6;
       let url3 = Constant.API6;
       let httpRequest3 = http.createHttp();
       let promise3 = httpRequest3.request(
         url3,
         {
           method: http.RequestMethod.POST,
           //传参，index+1表示机器编号
           extraData: 'machineNumber=' + (i + 1).toString(),
           connectTimeout: 60000,
           readTimeout: 60000,
           header: {
             "Content-Type": "application/x-www-form-urlencoded"
           }
         });
       console.log("begpromise3  "+i)
       await promise3.then((data) => {
         if (data.responseCode === http.ResponseCode.OK) {
           //将读到的数据转换为对象
           let dataPre = JSON.parse(JSON.parse(JSON.stringify(data.result)))
           let k
           let bk:number[]=[]
           for (let j = 0;j < dataPre.length; j++) {
             //轴承编号也从0开始
             let BNum = Number(dataPre[j]["bearingNumber"]) - 1
             //console.log(BNum.toString())
             if (pre[i][BNum] == null) {
               //若为空则初始化
               pre[i][BNum] = []
               bk[BNum]=0
               k = 0
             }
             pre[i][BNum][bk[BNum]] = []
             pre[i][BNum][bk[BNum]][0] = JSON.parse(JSON.stringify(dataPre[j]["predictSN"]))
             pre[i][BNum][bk[BNum]][1] = JSON.parse(JSON.stringify(dataPre[j]["machineNumber"]))
             pre[i][BNum][bk[BNum]][2] = JSON.parse(JSON.stringify(dataPre[j]["bearingNumber"]))
             pre[i][BNum][bk[BNum]][3] = JSON.parse(JSON.stringify(dataPre[j]["rul"]))
             pre[i][BNum][bk[BNum]][4] = JSON.parse(JSON.stringify(dataPre[j]["predictTime"]))
             bk[BNum]++
           }
         }
       })
       
       console.log("promise3  "+i+"over")
     }

     console.log("beggo")
     if(f==0){
       console.log("go to table")
       router.replaceUrl({
         url:'pages/Table',
         params:{
           //ipAddress:ipAddress
         }
       }, router.RouterMode.Single)
     }
     else if(f==1){
       router.replaceUrl({
         url:'pages/detail',
         params:{
           machine: mn,
           bear: bn,
           //ipAddress:ipAddress
         }
       }, router.RouterMode.Single)
     }

  }

  build(){
    Column(){
      Blank();
      Blank();
      Column(){
        Text('加载中').fontColor(Constant.LOGIN_COLOR).fontSize(36).fontWeight(FontWeight.Bold)
      }
      .alignItems(HorizontalAlign.Start).opacity(this.opacityValue)
      .animation({
        iterations:1
      })
      Blank();
      Blank();
      Blank();
    }.width('100%').height('100%').linearGradient({
      colors:[
        [Constant.loa_col_0,0],[Constant.loa_col_1,1]
      ]
    })
  }
}
```
&emsp;&emsp;这里我专门设计一个界面进行数据加载，主要是方便进行刷新等操作。

## Table
```typescript
import { dataList, mbNum,pre } from './load'
import Constant from '../common/Constant'
import router from '@ohos.router';

let zero:number[] = new Array(100).fill(0);
@Entry
@Component
struct Table{

  private classifyScroller : Scroller=new Scroller()//一级列表Scroller
  private scroller : Scroller=new Scroller()//二级列表Scroller
  @State currentTagIndex:number=0;
  private tmp:Number=0;
  @State isClickTagList:Boolean = false;//是否点击一级列表

  @State data:string[][][][]=dataList
  @State mb:number[]=mbNum
  @State preData:String[][][][]=pre
  machineNumber:string[]=[]
  bearNumber:string[]=[]
  record:number[]=[]
  //data_list:string[][][][]=[]

  findClassIndex(index:number){
    let ans=0
    for(let i=0;i<this.record.length;i++){
      if(index>=this.record[i]&&index<this.record[i+1]){
        ans=i;
        break;
      }
    }
    return ans
  }

  findItemIndex(index:number){
    return this.record[index]
  }

  aboutToAppear(){
    console.log("arrival table")
    this.record.push(0)
    for(let i=1;i<=this.mb.length;i++){
      this.record[i]=this.record[i-1]+this.mb[i-1]
    }
    for(let i=0;i<this.mb.length;i++){
      this.machineNumber[i]='machine_'+(i+1).toString()
      for(let j=0;j<this.mb[i];j++){
        this.bearNumber.push('machine_'+(i+1).toString()+"_bearing_"+(j+1).toString())
      }
    }
  }

  build(){
    Column(){
      Blank().height("2%").width('100%').backgroundColor('#ffffff')
      Row(){

        Text('ArgBearing').fontColor(Constant.LOGIN_COLOR).fontSize(30).fontWeight(FontWeight.Bold).margin({left:15}).fontStyle(FontStyle.Italic)

        Image($r('app.media.ic_public_refresh'))
          .interpolation(ImageInterpolation.High)
          .height("50%")
          .margin({left:500})
          .backgroundColor(Color.White)
          .onClick(()=>{
            router.replaceUrl({
              url: 'pages/load',
              params: {
                f:0,
                mn:-1,
                bn:-1,
                //ipAddress:ipAddress
              }
            }, router.RouterMode.Single)
          })
        Image($r('app.media.ic_public_more_filled'))
          .interpolation(ImageInterpolation.High)
          .height("60%")
          .margin({left:20})
          .backgroundColor(Color.White)
          .onClick(()=>{
            router.pushUrl({
              url: 'pages/Setting',
              params: {
                //ipAddress:ipAddress
              }
            }, router.RouterMode.Single)
          })
        Blank()
      }.backgroundColor(Color.White).height("5%").width('100%')
      Row(){
        List({scroller:this.classifyScroller,initialIndex:0}){
          ForEach(this.machineNumber,(item:string,index:number)=>{
            ListItem(){
              Column(){
                Row(){
                  Text().width(5).height(60).backgroundColor(Constant.LOGIN_COLOR)
                    .visibility(this.currentTagIndex===index?Visibility.Visible:Visibility.None)
                  Column(){
                    Text(item).width('100%').height('100%').fontSize(25)
                      .fontWeight(this.currentTagIndex===index?Constant.LOGIN_COLOR:'#333333')
                      .textAlign(TextAlign.Center)
                  }

                }
                .backgroundColor(this.currentTagIndex===index?Color.White:'#f8f8f8')
              }.height('15%')
              .onClick(()=>{
                this.currentTagIndex=index
                this.classifyScroller.scrollToIndex(index)
                let itemIndex = this.findItemIndex(index)
                this.scroller.scrollToIndex(itemIndex)
              })
            }
          })
        }
        .onScrollIndex((start:number,end:number)=>{
          this.currentTagIndex=start
          this.classifyScroller.scrollToIndex(start)
          let itemIndex = this.findItemIndex(start)
          this.scroller.scrollToIndex(itemIndex)
        })
        .listDirection(Axis.Vertical)
        .scrollBar(BarState.Off)
        .height('93%')
        .width('27%')

        List({scroller:this.scroller,space:0}){
          ForEach(this.bearNumber,(item:string,index:number)=>{
            ListItem(){
              Column() {
                Row({ space: 10 }) {
                    if (this.data[this.findClassIndex(index)][index-this.record[this.findClassIndex(index)]][this.data[this.findClassIndex(index)][index-this.record[this.findClassIndex(index)]].length-1][3] === "0") {
                      Image($r('app.media.bear_g'))
                        .aspectRatio(1)
                        .objectFit(ImageFit.Contain)
                        .height("75%")
                        .backgroundColor(Color.White)
                        .margin({left:20})
                    }
                    else if (this.data[this.findClassIndex(index)][index-this.record[this.findClassIndex(index)]][this.data[this.findClassIndex(index)][index-this.record[this.findClassIndex(index)]].length-1][3] === "3") {
                      Image($r('app.media.bear_r'))
                        .aspectRatio(1)
                        .objectFit(ImageFit.Contain)
                        .height("75%")
                        .backgroundColor(Color.White)
                        .margin({left:20})
                    }
                    else {
                      Image($r('app.media.bear_y'))
                        .aspectRatio(1)
                        .objectFit(ImageFit.Contain)
                        .height("75%")
                        .backgroundColor(Color.White)
                        .margin({left:20})
                    }
                  //}
                  Column() {
                    Text(item).height("30%").fontSize(23).width("45%")

                    Text('剩余寿命(RUL)：'+this.preData[this.findClassIndex(index)][index-this.record[this.findClassIndex(index)]][this.preData[this.findClassIndex(index)][index-this.record[this.findClassIndex(index)]].length-1][3].substring(0,5)+" 天").height("30%").fontSize(20)

                  }.alignItems(HorizontalAlign.Start)
                  .justifyContent(FlexAlign.SpaceEvenly)
                  .height('100%').margin({left:20})
                  Button("详情",{type:ButtonType.Circle}).height("70").margin({left:80,right:50})
                    .onClick(() => {
                      router.pushUrl({
                        url: 'pages/detail',
                        params: {
                          machine: this.findClassIndex(index),
                          bear: index - this.record[this.findClassIndex(index)],
                          //ipAddress:ipAddress
                        }
                      }, router.RouterMode.Single)
                    })
                }.height(120).backgroundColor(Color.White)
              }
              .justifyContent(FlexAlign.SpaceBetween)
            }
            .onClick(()=>{
              let currentClassIndex=this.findClassIndex(index)
              if(currentClassIndex!=this.tmp){
                this.tmp=currentClassIndex
                this.currentTagIndex=currentClassIndex
                this.classifyScroller.scrollToIndex(currentClassIndex)
              }
            })
          })
        }
        .height('93%')
        .width('73%')
        .onScrollIndex((start)=>{
          let currentClassIndex=this.findClassIndex(start)
          if(currentClassIndex!=this.tmp){
            this.tmp=currentClassIndex
            this.currentTagIndex=currentClassIndex
            this.classifyScroller.scrollToIndex(currentClassIndex)
          }
        })
      }
    }
    .height("100%").width("100%")
    .backgroundColor('#f8f8f8')
  }
}
```
&emsp;&emsp;这里我们主要通过一个二级列表，实现对轴承和机器的归纳管理。

## Detail
```typescript
import Constant from '../common/Constant'
import router from '@ohos.router'
import { dataList, mbNum,pre } from './load'
import http from '@ohos.net.http';
// import {PullToRefresh} from '@ohos/pulltorefresh'

let machineN=router.getParams()['machine']
let bearN = router.getParams()['bear']
//let ipAddress = router.getParams()['ipAddress']

@Entry
@Component
struct detail{

  @State data:string[][][][]=dataList
  @State mb:number[]=mbNum;
  @State pre:string[][][][]=pre
  b_index:number[]=[]
  p_index:number[]=[]
  dataNew:string[]=[]

  private scroller:Scroller=new Scroller()

  @State isExpanded: boolean = false;
  @State isDet:boolean=true;
  @State isRefreshing:boolean=false;

  aboutToAppear(){
    let j=0
    for(let i=this.data[machineN][bearN].length-1;i>=0;i--){
      this.b_index[j]=i
      j++
    }

    let k=0
    for(let i=this.pre[machineN][bearN].length-1;i>=0;i--){
      this.p_index[k]=i
      k++
    }

  }

  private getLoc(a:number):string{
    if(a==0) return "滚珠"
    else if(a==1) return "内圈"
    else if(a==2) return "外圈3点钟"
    else return "外圈6点钟"
  }

  openDialog(text: string[]) {
  AlertDialog.show(
    {
      //title: 'title',
      message: text[0]+"\n"+text[1],
      autoCancel: true,
      alignment: DialogAlignment.Bottom,
      offset: { dx: 0, dy: -20 },
    }
  )
}
  build(){
    Column() {
      Blank().height("2%").width('100%').backgroundColor('#ffffff')
      Row() {
        Image($r('app.media.ic_public_left_filled'))
          .interpolation(ImageInterpolation.High)
          .height("60%")
          .margin({ left: 15 })
          .backgroundColor(Color.White)
          .onClick(() => {
            router.replaceUrl({
              url: 'pages/Table',
              params: {
                //ipAddress: ipAddress
              }
            }, router.RouterMode.Single)
          })
        Text('Machine ' + (Number(machineN) + 1).toString() + ' ' + 'bearing ' + (Number(bearN) + 1))
          .fontColor(Constant.LOGIN_COLOR)
          .fontWeight(FontWeight.Bold)
          .fontSize(20)
          .width("50%")
          .padding(10)
        Image($r('app.media.ic_public_refresh'))
          .interpolation(ImageInterpolation.High)
          .height("50%")
          .backgroundColor(Color.White)
          .margin({ left: 240 })
          .onClick(() => {
            router.replaceUrl({
              url: 'pages/load',
              params: {
                f: 1,
                mn: machineN,
                bn: bearN,
                //ipAddress: ipAddress
              }
            }, router.RouterMode.Single)
          })
        Image($r('app.media.ic_public_more_filled'))
          .interpolation(ImageInterpolation.High)
          .height("60%")
          .backgroundColor(Color.White)
          .margin({ left: 20 })
          .onClick(() => {
            router.pushUrl({
              url: 'pages/explain',
              params: {}
            }, router.RouterMode.Single)
          })
      }.width('100%')
      .backgroundColor(Color.White).height("5%")

      if (this.data[machineN][bearN][this.data[machineN][bearN].length-1][3] === "0") {
        Image($r('app.media.bearing_5')).height("33%")
      }
      else {
        if (this.data[machineN][bearN][0][4] === "0")
          Image($r('app.media.bearing_0')).height("33%")
        else if (this.data[machineN][bearN][0][4] === "1")
          Image($r('app.media.bearing_1')).height("33%")
        else if (this.data[machineN][bearN][0][4] === "2")
          Image($r('app.media.bearing_2')).height("33%")
        else
          Image($r('app.media.bearing_3')).height("33%")
      }

      Row(){
        Button('故障检测', { type: ButtonType.Normal, stateEffect: false })
          .width('50%')
          .height("100%")
          .fontSize(16)
          .fontWeight(FontWeight.Medium)
          .backgroundColor(this.isDet?'#0166e6':'#0171FF')
          .onClick(()=>{
            this.isDet=true
          })
        Button('寿命预测', { type: ButtonType.Normal, stateEffect: false })
          .width('50%')
          .height("100%")
          .fontSize(16)
          .fontWeight(FontWeight.Medium)
          .backgroundColor(this.isDet?'#0171FF':'#0166e6')
          .onClick(()=>{
            this.isDet=false
          })
      }.height("5%")

      if(this.isDet) {

        List({ space: 10 }) {
          ForEach(this.b_index, (item: number, index: number) => {
            ListItem() {
              Column() {
                Row() {
                  if (this.data[machineN][bearN][item][3] === "0") {
                    Image($r('app.media.bear_g'))
                      .aspectRatio(1)
                      .objectFit(ImageFit.Contain)
                      .height(80)
                      .backgroundColor(Color.White)
                      .align(Alignment.Center)
                      .width('25%')
                      .margin({ left: 60 })
                  }
                  else if (this.data[machineN][bearN][item][3] === "3") {
                    Image($r('app.media.bear_r'))
                      .aspectRatio(1)
                      .objectFit(ImageFit.Contain)
                      .height(80)
                      .backgroundColor(Color.White)
                      .align(Alignment.Center)
                      .width('25%')
                      .margin({ left: 60 })
                  }
                  else {
                    Image($r('app.media.bear_y'))
                      .aspectRatio(1)
                      .objectFit(ImageFit.Contain)
                      .height(80)
                      .backgroundColor(Color.White)
                      .align(Alignment.Center)
                      .width('25%')
                      .margin({ left: 60 })
                  }
                  Column() {
                    Text('detectSN: ' + this.data[machineN][bearN][item][0])
                      .textAlign(TextAlign.Start)
                      .width('100%')
                      .height(40)
                      .fontSize(20)
                    Text('detectTime:' + this.data[machineN][bearN][item][12])
                      .textAlign(TextAlign.Start)
                      .width('100%')
                      .height(40)
                      .fontSize(20)
                  }.width('70%').margin({ left: 50 })
                }.width('100%').height(80)

                if (this.isExpanded) {
                  Row() {
                    Column() {
                      if (this.data[machineN][bearN][item][3] === "0") {
                        Image($r('app.media.bearing_5')).height(120).margin({ left: 20 })
                      }
                      else {
                        if (this.data[machineN][bearN][item][4] === "0")
                          Image($r('app.media.bearing_0')).height(120).margin({ left: 20 })
                        else if (this.data[machineN][bearN][item][4] === "1")
                          Image($r('app.media.bearing_1')).height(120).margin({ left: 20 })
                        else if (this.data[machineN][bearN][item][4] === "2")
                          Image($r('app.media.bearing_2')).height(120).margin({ left: 20 })
                        else
                          Image($r('app.media.bearing_3')).height(120).margin({ left: 20 })
                      }
                    }

                    Column() { //文本
                      Row() {
                        Text('loc1: ' + this.getLoc(Number(this.data[machineN][bearN][item][4])))
                          .width('25%')
                          .textAlign(TextAlign.Start)
                          .height(40)
                          .fontSize(20)

                        Text('Dia1: ' + Number(this.data[machineN][bearN][item][3]) * 7 + " mils")
                          .width('24%')
                          .textAlign(TextAlign.Start)
                          .height(40)
                          .fontSize(20)

                        Text('Score1: ' + this.data[machineN][bearN][item][5].substring(0, 6))
                          .width('40%')
                          .textAlign(TextAlign.Start)
                          .height(40)
                          .fontSize(20)
                        Blank().width('10%')
                      }.width('100%').height(40)

                      Row() {
                        Text('loc2: ' + this.getLoc(Number(this.data[machineN][bearN][item][7])))
                          .width('25%')
                          .textAlign(TextAlign.Start)
                          .height(40)
                          .fontSize(20)

                        Text('Dia2: ' + Number(this.data[machineN][bearN][item][6]) * 7 + " mils")
                          .width('24%')
                          .textAlign(TextAlign.Start)
                          .height(40)
                          .fontSize(20)

                        Text('Score2: ' + this.data[machineN][bearN][item][8].substring(0, 6))
                          .width('40%')
                          .textAlign(TextAlign.Start)
                          .height(40)
                          .fontSize(20)
                        Blank().width('10%')
                      }.width('100%').height(40)

                      Row() {
                        Text('loc3: ' + this.getLoc(Number(this.data[machineN][bearN][item][10])))
                          .width('25%')
                          .textAlign(TextAlign.Start)
                          .height(40)
                          .fontSize(20)

                        Text('Dia3: ' + Number(this.data[machineN][bearN][item][9]) * 7 + " mils")
                          .width('24%')
                          .textAlign(TextAlign.Start)
                          .height(40)
                          .fontSize(20)

                        Text('Score3: ' + this.data[machineN][bearN][item][11].substring(0, 6))
                          .width('40%')
                          .textAlign(TextAlign.Start)
                          .height(40)
                          .fontSize(20)
                        Blank().width('10%')
                      }.width('100%').height(40)
                    }.margin({ left: 5 })
                  }
                }
              }
              .width('100%')
              .height(this.isExpanded ? 200 : 80)
              .onClick(() => {
                this.isExpanded = !this.isExpanded;
              })
              .borderRadius(20)
              .backgroundColor(Color.White)
            }
          }, item => item)


        }.height("50%")

        Column() {
          Blank().height("10%")
          Button('立 即 检 测', { type: ButtonType.Capsule, stateEffect: true })
            .width('90%')
            .height("80%")
            .fontSize(16)
            .fontWeight(FontWeight.Medium)
            .backgroundColor('#007DFF')
            .onClick(() => {
              let httpRequest7 = http.createHttp();
              let url7 = Constant.API7;
              let promise7 = httpRequest7.request(
                url7,
                {
                  method: http.RequestMethod.POST,
                  extraData:'machine_id='+Number(machineN+1)+'&bearing_id='+Number(bearN+1),
                  connectTimeout: 60000,
                  readTimeout: 60000,
                  header: {
                    "Content-Type": "application/x-www-form-urlencoded"
                  }
                });
              promise7.then((data) => {
                if (data.responseCode === http.ResponseCode.OK) {
                  let dataGet=JSON.parse(JSON.parse(JSON.stringify(data.result)))
                  this.dataNew[0]="Machine_ID: "+JSON.parse(JSON.stringify(dataGet["machine_id"]))
                  this.dataNew[1]="Bearing_ID: "+JSON.parse(JSON.stringify(dataGet["bearing_id"]))
                  this.dataNew[2]="Loc1: "+this.getLoc(Number(JSON.stringify(dataGet["faultLoc1"])))
                  this.dataNew[3]="Dia1: "+Number(JSON.stringify(dataGet["faultDia1"]))*7+" mils"
                  this.dataNew[4]="Score1: "+JSON.stringify(dataGet["faultScore1"]).substring(0,6)
                  this.dataNew[5]="Loc2: "+this.getLoc(Number(JSON.stringify(dataGet["faultLoc2"])))
                  this.dataNew[6]="Dia2: "+Number(JSON.stringify(dataGet["faultDia2"]))*7+" mils"
                  this.dataNew[7]="Score2: "+JSON.stringify(dataGet["faultScore2"]).substring(0,6)
                  this.dataNew[8]="Loc3: "+this.getLoc(Number(JSON.stringify(dataGet["faultLoc3"])))
                  this.dataNew[9]="Dia3: "+Number(JSON.stringify(dataGet["faultDia3"]))*7+" mils"
                  this.dataNew[10]="Score3: "+JSON.stringify(dataGet["faultScore3"]).substring(0,6)

                  console.info('Result:' + data.result);
                  console.info('code:' + data.responseCode);
                  AlertDialog.show(
                    {
                      title: '检测结果', // 标题
                      message: this.dataNew[0]+"\n"+this.dataNew[1]+"\n"+this.dataNew[2]+"\n"+this.dataNew[3]+"\n"+this.dataNew[4]+"\n"+this.dataNew[5]+"\n"+this.dataNew[6]+"\n"+this.dataNew[7]+"\n"+this.dataNew[8]+"\n"+this.dataNew[9]+"\n"+this.dataNew[10], // 内容
                      autoCancel: false, // 点击遮障层时，是否关闭弹窗。
                      alignment: DialogAlignment.Center, // 弹窗在竖直方向的对齐方式
                      offset: { dx: 0, dy: -20 }, // 弹窗相对alignment位置的偏移量
                      // gridCount:5 ,
                      primaryButton: {
                        value: '确定',
                        action: () => {
                          console.info('Callback when the first button is clicked');
                          router.replaceUrl({
                            url: 'pages/load',
                            params: {
                              f: 1,
                              mn: machineN,
                              bn: bearN,
                              //ipAddress: ipAddress
                            }
                          }, router.RouterMode.Single)
                        }
                      },
                      cancel: () => { // 点击遮障层关闭dialog时的回调
                        console.info('Closed callbacks');
                      }
                    }
                  )
                }
              }).catch((err) => {
                console.info('error:' + JSON.stringify(err));
              });
            })
          Blank().height("10%")
        }.height("5%").width("100%")

      }
      else{

        List({ space: 10 }) {
          ForEach(this.p_index, (item: number, index: number) => {
            ListItem() {
              Column() {
                Text('predictSN: ' + this.pre[machineN][bearN][item][0])
                  .width('100%')
                  .textAlign(TextAlign.Start)
                  .height(40)
                  .fontSize(20)
                  .margin({left:50})
                Text('剩余寿命(RUL): ' + this.pre[machineN][bearN][item][3].substring(0,5)+" 天")
                  .width('100%')
                  .textAlign(TextAlign.Start)
                  .height(40)
                  .fontSize(20)
                  .margin({left:50})
                Text('predictTime: ' + this.pre[machineN][bearN][item][4])
                  .width('100%')
                  .textAlign(TextAlign.Start)
                  .height(40)
                  .fontSize(20)
                  .margin({left:50})
              }
              .width('100%')
              .height(120)
              .borderRadius(20)
              .backgroundColor(Color.White)
            }
          }, item => item)
        }.height("50%")

        Column() {
          Blank().height("10%")
          Button('立 即 预 测', { type: ButtonType.Capsule, stateEffect: true })
            .width('90%')
            .height("80%")
            .fontSize(16)
            .fontWeight(FontWeight.Medium)
            .backgroundColor('#007DFF')
            .onClick(() => {
              let httpRequest8 = http.createHttp();
              let url8 = Constant.API8;
              let promise8 = httpRequest8.request(
                url8,
                {
                  method: http.RequestMethod.POST,
                  extraData:'machine_id='+Number(machineN+1)+'&bearing_id='+Number(bearN+1),
                  connectTimeout: 60000,
                  readTimeout: 60000,
                  header: {
                    "Content-Type": "application/x-www-form-urlencoded"
                  }
                });
              promise8.then((data) => {
                if (data.responseCode === http.ResponseCode.OK) {
                  let dataGet=JSON.parse(JSON.parse(JSON.stringify(data.result)))
                  this.dataNew[0]="Machine_ID: "+JSON.parse(JSON.stringify(dataGet["machine_id"]))
                  this.dataNew[1]="Bearing_ID: "+JSON.parse(JSON.stringify(dataGet["bearing_id"]))
                  this.dataNew[2]="剩余寿命(RUL): "+JSON.stringify(dataGet["rul"]).substring(0,5)+" 天"

                  console.info('Result:' + data.result);
                  console.info('code:' + data.responseCode);

                  AlertDialog.show(
                    {
                      title: '预测结果', // 标题
                      message: this.dataNew[0]+"\n"+this.dataNew[1]+"\n"+this.dataNew[2], // 内容
                      autoCancel: false, // 点击遮障层时，是否关闭弹窗。
                      alignment: DialogAlignment.Center, // 弹窗在竖直方向的对齐方式
                      offset: { dx: 0, dy: -20 }, // 弹窗相对alignment位置的偏移量
                      primaryButton: {
                        value: '确定',
                        action: () => {
                          console.info('Callback when the first button is clicked');
                          router.replaceUrl({
                            url: 'pages/load',
                            params: {
                              f: 1,
                              mn: machineN,
                              bn: bearN,
                              //ipAddress: ipAddress
                            }
                          }, router.RouterMode.Single)
                        }
                      },
                      cancel: () => { // 点击遮障层关闭dialog时的回调
                        console.info('Closed callbacks');
                      }
                    }
                  )
                }
              }).catch((err) => {
                console.info('error:' + JSON.stringify(err));
              });
            })
          Blank().height("10%")
        }.height("5%").width("100%")
      }
      // .divider({
      //   strokeWidth: 1,
      //   startMargin: 60,
      //   endMargin: 10,
      //   color: Constant.LOGIN_COLOR
      // })
    }
    .backgroundColor('#f8f8f8')
  }
}
```
&emsp;&emsp;我们通过状态变量isExpanded，isDet来实现不同功能的切换。


&emsp;&emsp;综上，我们就实现了一个比较简单的Harmony APP。
