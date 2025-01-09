[合集 \- rustGUI\-iced\-V0\.13(4\)](https://github.com)[1\.\[rustGUI]\[iced]基于rust的GUI库iced（0\.13）的部件学习（00）：iced简单窗口的实现以及在窗口显示中文2024\-12\-31](https://github.com/rongjv/p/18641486/rongjv1990)[2\.\[rustGUI]\[iced]基于rust的GUI库iced（0\.13）的部件学习（01）：为窗口设置布局（column、row）01\-08](https://github.com/rongjv/p/18644157):[slowerssr加速器官网](https://secine.com/)[3\.\[rustGUI]\[iced]基于rust的GUI库iced（0\.13）的部件学习（02）：滑动条部件实现部件（文本或其他）缩放（slider）01\-08](https://github.com/rongjv/p/18659437)4\.\[rustGUI]\[iced]基于rust的GUI库iced（0\.13）的部件学习（03）：图像的导入、显示、调整（暨image部件的使用介绍）01\-09收起
**前言**
本文是关于iced库的部件介绍，iced库是基于rust的GUI库，作者自述是受Elm启发。
iced目前的版本是0\.13\.1，相较于此前的0\.12版本，有较大改动。
本合集是基于新版本的关于分部件（widget）的使用介绍，包括源代码介绍、实例使用等。


**环境配置**
系统：window10
平台：visual studio code
语言：rust
库：iced 0\.13


## 图像导入及显示


iced中显示图像，可以使用image部件，但image部件不是默认启用的，需要启用feature。在**toml**文件添加：



```
iced={version="0.13.1", features=["image"]}

```

然后在代码中导入：



```
use iced::widget::{button,column,text,row,container,slider,image};

```

之前我们在介绍为iced窗口设置图标时还提到过一个外部图像库image：



```
image="0.25.5"

```

导入image，为了不与iced的image部件名称冲突，需要为image改个名称：



```
extern crate image as eximg;

```

#### 1、图像导入


![image](https://img2024.cnblogs.com/blog/2558709/202501/2558709-20250109091648557-2041027323.png)


图像导入，通常需要使用文件对话框来导入文件，在iced中并没有提供对应的部件，因此，我们需要使用一个外部库，RFD，首先在toml中添加：



```
rfd="0.15.2"

```

下面是rfd库支持的功能：
![image](https://img2024.cnblogs.com/blog/2558709/202501/2558709-20250109091741283-16995446.png)
包括了文件导入、保存、文件夹选择等，当然rfd也可以支持消息对话框。



```
use rfd::{FileDialog,MessageDialog};

```

本文的目的是实现图像文件的导入与显示，并可以调整一些图像属性，如大小、透明度等，因此，我们将添加一个按钮用于触发文件选中对话框，获取图像文件的路径并显示。
我们为结构体添加一个变量用于记录图像路径：



```
#[derive(Clone)]
struct Counter {
    slidervalue:f32,
    slidervalue2:f32,
    imgpath:String,
}

```

上述imgpath即图像路径变量，另外两个变量用于接受slider部件的实时值，因为我们需要调整图像的大小和透明度，正好可以适应slider，而关于slider的使用介绍，我们在上一篇博文里有单独介绍：
[\[rustGUI]\[iced]基于rust的GUI库iced（0\.13）的部件学习（02）：滑动条部件实现部件（文本或其他）缩放（slider）](https://github.com "https://github.com/rongjv/p/18659437")
然后我们添加按钮触发消息：



```
#[derive(Debug,Clone, Copy)]
enum Message {
    SliderChange(f32),
    SliderChange2(f32),
    BtnLoad,
}

```

上述的BtnLoad是按钮触发消息，另外两个是slider的滑动时触发，就不多说了。
我们希望的是，点击按钮时弹出文件对话框，因此需要在view函数里添加：



```
let btn_load=button(text("加载图片").size(15)).width(80).height(40)
                    .on_press(Message::BtnLoad);

```

然后在update函数里更新状态：



```
Message::BtnLoad =>{
                if let Some(file)=FileDialog::new()
                            .set_title("选择图片")
                            .add_filter("Image",&["png","jpg","jpeg","bmp"])
                            .add_filter("png",&["png"])
                            .set_directory("C://")
                            .pick_file(){
                                
                                println!("当前选择的文件：{:?}",file);
                                self.imgpath=file.display().to_string();

                            };
                
            }

```

上述代码中，**FileDialog::new()**是rfd库中文件夹选择的用法，看rfd的官方示例：
![image](https://img2024.cnblogs.com/blog/2558709/202501/2558709-20250109092934147-227106748.png)
我们在实例中，将获取的图像路径传递给我们设置的变量imgpath。


#### 2、图像显示


当我们获取到了图像的本地路径之后，就可以使用iced的image部件来显示图像，我们首先来看下iced中image的定义：


###### 官方源码



```
#[derive(Debug)]
pub struct Image {
    handle: Handle,
    width: Length,
    height: Length,
    content_fit: ContentFit,
    filter_method: FilterMethod,
    rotation: Rotation,
    opacity: f32,
}

```

我们主要关注的就是handle这个参数，它的类型是Handle，可以理解为图像的原始数据来源，Handle在iced中的定义：


###### 官方源码



```
pub enum Handle {
    /// A file handle. The image data will be read
    /// from the file path.
    ///
    /// Use [`from_path`] to create this variant.
    ///
    /// [`from_path`]: Self::from_path
    Path(Id, PathBuf),

    /// A handle pointing to some encoded image bytes in-memory.
    ///
    /// Use [`from_bytes`] to create this variant.
    ///
    /// [`from_bytes`]: Self::from_bytes
    Bytes(Id, Bytes),

    /// A handle pointing to decoded image pixels in RGBA format.
    ///
    /// Use [`from_rgba`] to create this variant.
    ///
    /// [`from_rgba`]: Self::from_rgba
    Rgba {
        /// The id of this handle.
        id: Id,
        /// The width of the image.
        width: u32,
        /// The height of the image.
        height: u32,
        /// The pixels.
        pixels: Bytes,
    },
}

```

可以看到，Handle定义了三种获取图像原始数据的方式，一种是直接从图像路径获取，一种是图像的字节数组获取，一种是从rgba数据获取。
我们在本文显然是使用第一种方式，即图像路径来获取handle，用于显示。iced官方给了典型的image使用代码：



```
image("ferris.png")

```

这是最简单的应用，不过我们在实例使用时，因为希望能调整图像的大小与透明度，所以可以这样写：



```
let img_handle=image::Handle::from_path(self.imgpath.clone());
let img1=image(img_handle).opacity(opacity).width(img_w).height(img_h);

```

其中，opacity用于调整透明度，width和height用于调整图像尺寸。
我们来看下实际显示效果：
![image](https://img2024.cnblogs.com/blog/2558709/202501/2558709-20250109094054337-1607939196.png)


#### 3、图像调整


有了之前slider调整text文字大小的经验，此处就比较简单了，因为我们已经获取了图像路径，显然也就能获取图像尺寸了，这里，我们使用外部的image库来处理。image库的功能还是很强大的，看下它支持的图像处理格式：
![image](https://img2024.cnblogs.com/blog/2558709/202501/2558709-20250109094908236-663123206.png)
为了方便管理，我们将**获取图像尺寸**封装为一个函数，与此前的img\_to\_icon函数放到一起，使用时直接调用即可：



```
///
/// 获取图片大小
/// 
/// 例：jpg or png -> (w,h)
pub fn get_img_size(path:&str) ->(f32,f32){

    if path != "" {
        let img_re=eximg::open(path);
        match img_re{
            Ok(img)=>{
                return (img.width() as f32,img.height() as f32)
            },
            Err(e)=>{
                let res=MessageDialog::new()
                        .set_title("错误")
                        .set_level(rfd::MessageLevel::Error)
                        .set_buttons(rfd::MessageButtons::Ok)
                        .set_description(&e.to_string())
                        .show();
                if res == rfd::MessageDialogResult::Yes{
                    return (0.0,0.0)
                }
                return (0.0,0.0)
            }
        }
        //return  (img.width() as f32,img.height() as f32)
    }
    return (0.0,0.0)

}

```


```
 let img_size=imgprocess::get_img_size(&self.imgpath);

```

看下完整的view函数：



```
let btn_load=button(text("加载图片").size(15)).width(80).height(40)
                   .on_press(Message::BtnLoad);
       let sl1=slider(
           0.0..=1.0, 
           self.slidervalue, 
           Message::SliderChange)
                               .height(40).width(200)
                               .step(0.01);
      
       let opacity= self.slidervalue;
       let tt1=text("透明度:").size(15);
       let tt2=text(format!("{:.2}",opacity)).size(15);
       let row1=row![tt1,sl1,tt2,].spacing(10)
                                               .align_y(iced::Center);


       let slider_scale=slider(0.1..=2.0,self.slidervalue2,Message::SliderChange2)
                                                       .height(40).width(200)
                                                       .step(0.01);
       let tt3=text("缩放:").size(15);
       let tt4=text(format!("{:.2}",self.slidervalue2)).size(15);
       let row2=row![tt3,slider_scale,tt4,].spacing(10);

       let tt_imgpath=text("当前图片路径:").size(15);
       let tt_imgpath2=text(format!("{}",self.imgpath)).size(15);
       let row3=row![tt_imgpath,tt_imgpath2,].spacing(10);

       let img_handle=image::Handle::from_path(self.imgpath.clone());
       let img_size=imgprocess::get_img_size(&self.imgpath);
       let img_w=img_size.0 * self.slidervalue2;
       let img_h=img_size.1 * self.slidervalue2;
       let tt_imgsize=text(format!("图片大小:{:?}",img_size)).size(15);
       //let row4=row![].spacing(10);

       let img1=image(img_handle).opacity(opacity).width(img_w).height(img_h);
       //let cont_color=Color::from_rgb(120.0, 120.0, 0.0);
       let cont_color2=iced::color!(0xE9E7E7,0.5);//#E9E7E7FF
       let cont_img=container(img1)
               .width(1000).height(1000)
               .align_x(iced::Center).align_y(iced::Center)
               .style(move |t|styles::mycontainerstyle(t, cont_color2));
       

       column![
           btn_load,
           row1,
           row2,
           row3,
           tt_imgsize,
           cont_img,
       ].align_x(iced::Center)
       .padding(20)
       .into()

```

它整体显示效果如下：
![image](https://img2024.cnblogs.com/blog/2558709/202501/2558709-20250109095730611-1770061744.png)


#### 4、综述


先来看看动态演示：
![image](https://img2024.cnblogs.com/blog/2558709/202501/2558709-20250109101153890-1943265299.gif)
结合前文slider的使用，本文介绍了如何在iced中导入图像、显示图像以及调整图像，当然也是很简单的示例。比如导入按钮，后续将会使用菜单替代。


