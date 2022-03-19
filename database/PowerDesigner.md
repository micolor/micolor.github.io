## PowerDesigner显示注释

PowerDesigner默认显示的列是Name及类型，如下图示：

![img](https://img-blog.csdn.net/20170912150429751?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGlmZmZhdGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

现在需要显示注释列，以便使得ER图更加清晰。但是PowerDesigner勾选Comment显示没有效果，所以通过以下几步来处理：

双击表，弹出表属性对话框，切到ColumnTab，默认是没显示Comment的，显示Comment列，这么做

![img](https://img-blog.csdn.net/20170912150629182?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGlmZmZhdGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

设置显示Comment

![img](https://img-blog.csdn.net/20170912150611091?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGlmZmZhdGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

有了Comment列，并补充Comment信息

![img](https://img-blog.csdn.net/20170912150800908?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGlmZmZhdGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

确定保存，打开菜单 Tools>Display Perferences..

![img](https://img-blog.csdn.net/20170912150912448?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGlmZmZhdGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

调整显示的Attribute

![img](https://img-blog.csdn.net/20170912150937938?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGlmZmZhdGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

OK，保存，确定，退出设置页，应用到所有标识，可以看到表变化

![img](https://img-blog.csdn.net/20170912151036549?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGlmZmZhdGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Option   Explicit
ValidationMode   =   True
InteractiveMode   =   im_Batch
Dim blankStr
blankStr   =   Space(1)
Dim   mdl   '   the   current   model

'   get   the   current   active   model
Set   mdl   =   ActiveModel
If   (mdl   Is   Nothing)   Then
      MsgBox   "There   is   no   current   Model "
ElseIf   Not   mdl.IsKindOf(PdPDM.cls_Model)   Then
      MsgBox   "The   current   model   is   not   an   Physical   Data   model. "
Else
      ProcessFolder   mdl
End   If

Private   sub   ProcessFolder(folder)
On Error Resume Next
      Dim   Tab   'running     table
      for   each   Tab   in   folder.tables
            if   not   tab.isShortcut   then
                  tab.name   =   tab.comment
                  Dim   col   '   running   column
                  for   each   col   in   tab.columns
                  if col.comment = "" or replace(col.comment," ", "")="" Then
                        col.name = blankStr
                        blankStr = blankStr & Space(1)
                  else
                        col.name = col.comment
                  end if
                  next
            end   if
      next

      Dim   view   'running   view
      for   each   view   in   folder.Views
            if   not   view.isShortcut   then
                  view.name   =   view.comment
            end   if
      next

      '   go   into   the   sub-packages
      Dim   f   '   running   folder
      For   Each   f   In   folder.Packages
            if   not   f.IsShortcut   then
                  ProcessFolder   f
            end   if
      Next
end   sub 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

打开菜单Tools>Execute Commands>Edit/Run Script.. 或者用快捷键 Ctrl+Shift+X

 

![img](https://img-blog.csdn.net/20170912151230945?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGlmZmZhdGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

执行完，可以看到第3列显示备注哈哈，效果如下

![img](https://img-blog.csdn.net/20170912152444615?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGlmZmZhdGU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

原理就是把显示name的列的值，替换成注释的值，所以下次如果调整comment，还有重新执行脚本，所以最好放在最后执行。