### DBNet数据导入

* 构造单个字典存储源数据
  * data.keys={img_path, label, img}
* 通过transform转换数据，在配置文件中修改
  * DecodeImage
  * IaaAugment
  * EastRandomCropData
  * NormalizeImage
  * ToCHWImage
  * **DetLabelEncode** 
  * **MakeBorderMap**
  * **MakeShrinkMap**
  * KeepKeys：[]