---
layout: post
title: VBA编程学习-子过程应用（排序）
comments: true
tags:
  - VBA
categories: 
  - 教程
date: 2022-03-02 11:20:02
---
过程指的就是VBA模块中的代码，或者宏。通过教程的学习，记录实现以下应用的过程：
通过活动工作簿中Sheet名称的字母顺序，按从小到大的顺序重新整理工作簿Sheet位置。
<!--more-->
## 分析应用的需求
通过分析目标，发现应用需要实现以下需求：
1. 活动工作簿按照Sheet名称字母的升序进行排序；
2. 应用可以比较方便地执行；
3. 可以在不打开用于编程的工作簿时使用程序；
4. 可以在任何工作簿中运行；
5. 捕获错误，不弹出任何VBA错误消息。

## 分析应用的问题
梳理与应用相关的信息，发现目前存在以下问题待解决：
1. Excel中没有对Sheet进行排序的命令；
2. 宏录制器无法用来录制Sheet的排序；
3. 排序需要对Sheet进行移动；
4. 需要知道活动工作簿中Sheet的数量；
5. 需要知道活动工作簿中所有Sheet的名称；
6. 应用能够在其他工作簿中运行。

## 问题拟解决的思路
根据目前存在的待解决问题，拟考虑通过以下方式处理：
1. 标识出活动工作簿；
2. 获取工作簿中所有Sheet的名称，存放到一组字符串类型的数组里；
3. 对数组进行升序排列；
4. 根据新排序的数组重新移动Sheet位置。

## 宏录制器辅助
在应用实现的过程中，可以通过宏录制器快速了解VBA语法，例如移动Sheet的程序代码。
通过宏录制器，记录将Sheet3的位置通过鼠标拖动到Sheet1之前的动作，查看代码，发现Move的用法：
    ```
    Sub MoveSheet()
        Sheets("Sheet3").Select
        Sheets("Sheet3").Move Before:=Sheets(1)
    End Sub
    ```

## 立即窗口辅助
在应用实现的过程中，还可以通过立即窗口，快速查看代码的运行结果，例如测试查阅的语句。
为获取工作簿中Sheet数量，通过查阅资料，发现集合的Count属性，在立即窗口测试：
    `?ActiveWorkbook.Sheets.Count`
得到测试结果，为实际Sheet的数量；
为获取工作簿中Sheet的名称，在立即窗口测试：
    `?ActiveWorkbook.Sheets(1).Name`
得到测试结果，为实际Sheet的名称；

## 子功能编写
### 遍历集合中的每个成员
使用For Each-Next结构实现遍历集合的动作：
    ```
    Sub Text()
        For Each sht in ActiveWorkbook.Sheets
            MsgBox sht.Name
        Next sht
    End Sub
    ```
代码弹出与活动工作簿中Sheet数量相同的消息框，并对应相应的Sheet名称。

### 将活动工作簿的Sheet名称放入数组
因为不知道活动工作簿的Sheet具体数量，可以先通过带空括号的Dim语句声明数组，之后使用ReDim语句重新定义数组的维数，使其等于实际的Sheet数量。将Sheet名称输入到SheetNames数组中：
    ```
    Sub SortSheets()
        Dim SheetNames() As String
        Dim i As Long
        Dim SheetCount As Long
    
        SheetCount = ActiveWorkbook.Sheets.Count
        ReDim SheetNames(1 To SheetCount)

        For i = 1 To SheetCount
            SheetNames(i) = ActiveWorkbook.Sheets(i).Name
            Debug.Print SheetNames(i)
        Next i
    End Sub
    ```
可在立即窗口中显示结果，为SheetNames数组元素的名称。

### 对数组的元素进行排序
排序算法互联网上有很多示例，例如通过冒泡排序对数组进行排序，即通过嵌套For-Next循环，比较相邻两个元素值，如果前一个元素值大于后一个元素值，则交换位置，重复多次之后，得到升序的元素值。
    ```
    Sub BubbleSort(List() As String)
        Dim First As Long, Last As Long
        Dim i As Long, j As Long
        Dim Temp As String
        First = LBound(List)
        Last = UBound(List)
        For i = First To Last - 1
            For j = i + 1 To Last
                If List(i) > List(j) Then
                    Temp = List(j)
                    List(j) = List(i)
                    List(i) = Temp
                End If
            Next j
        Next i
    End Sub
    ```
过程接收不确定元素数量的List一维数组，通过LBound和UBound函数确定数组下界和上界。

### 对数组排序
通过之前宏录制得到移动Sheet的代码，加上For-Next使其遍历每个工作表：
    ```
    For i = 1 To SheetCount
        ActiveWorkbook.Sheets(SheetNames(i)).Move Before:=ActiveWorkbook.Sheets(i)
    Next i
    ```

### 代码整理
通过整理上述代码，并添加注释行，使代码更便于阅读：
    ```
    Sub SortSheets()
    '按照字母升序移动活动工作簿的Sheet

        Dim SheetNames() As String
        Dim SheetCount As Long
        Dim i As Long
    
        '统计活动工作簿Sheet数量，并重新设定SheetNames数组元素数量
        SheetCount = ActiveWorkbook.Sheets.Count
        ReDim SheetNames(1 To SheetCount)
   
        '获取活动工作簿每个Sheet名称
        For i = 1 To SheetCount
            SheetNames(i) = ActiveWorkbook.Sheets(i).Name
        Next i
   
        '调用BubbleSort程序对SheetNames数组进行排序
        Call BubbleSort(SheetNames)

        '移动Sheet
        For i = 1 To SheetCount
            ActiveWorkbook.Sheets(SheetNames(i)).Move Before:=ActiveWorkbook.Sheets(i)
        Next i
    
    End Sub
    ```

## 功能测试
### 问题发现
加载其他工作簿进行测试，会发现诸多问题，例如：
1. 工作簿的Sheet在移动期间，屏幕需要更新，因此Sheet数量越多，排序时间越长；
2. 排序有字母大小写问题，大写字母的值会大于小写字母；
3. Excel没有可见工作簿窗口，宏会运行失败；
4. 工作簿结构受保护的话，移动Sheet命令会失败；
5. 排序后最后一个Sheet会变成活动工作表，会改变原来的Sheet活动状态；
6. 通过Ctrl+Break终端宏的运行，VBA会弹出错误消息；
7. 宏无法返回，误操作导致排序运行后，只能手动恢复。

### 问题修复
1. 可插入指令关闭屏幕的更新动作，在宏完成后，更新动作恢复：
    `Application.ScreenUpdating = False`
2. 通过UCase函数，把Sheet名称转换为大写字母，再进行排序：
    `If UCase(List(i)) > UCase(List(j)) Then`
3. 通过代码检查是否有活动工作簿，若没有，则退出过程：
    `If ActiveWorkbook Is Nothing Then Exit Sub`
4. 如果工作簿结构受到保护，应该显示弹框消息，让用户自己取消保护，而不是程序：
    ```
    If ActiveWorkbook.ProtectStructure Then
        MsgBox ActiveWorkbook.Name & " is protected.", vbCritical, "Cannot Sort Sheets."
        Exit Sub
    End If
    ```
5. 添加一个变量，记录原来的活动工作表，排序完成后重新激活：
    `Set OldActive = ActiveSheet`
    `OldActive.Activate`
6. 禁用Ctrl+Break组合键功能：
    `Application.EnableCancelKey = xlDisabled`
7. 添加确认弹窗，确认用户是否要移动Sheet位置：
    `If MsgBox("Sort the sheets in the active workbook?", vbQuestion + vbYesNo) <> vbYes Then Exit Sub`

## 功能实现
通过修复上述问题，完善代码后，所有程序过程如下：
    ```
    Sub SortSheets()
    '按照字母升序移动活动工作簿的Sheet

        Dim SheetNames() As String
        Dim i As Long
        Dim SheetCount As Long
        Dim OldActive As Object
    
        '判断是否存在活动工作簿，如果存在，统计Sheet数量
        If ActiveWorkbook Is Nothing Then Exit Sub ' No active workbook
        SheetCount = ActiveWorkbook.Sheets.Count
    
        '检查工作簿结构是否受到保护，
        If ActiveWorkbook.ProtectStructure Then
            MsgBox ActiveWorkbook.Name & " is protected.", vbCritical, "Cannot Sort Sheets."
            Exit Sub
        End If

        '用户确认是否需要进行Sheet排序
        If MsgBox("Sort the sheets in the active workbook?", vbQuestion + vbYesNo) <> vbYes Then Exit Sub

        '禁用Ctrl+Break组合键
        Application.EnableCancelKey = xlDisabled
       
        '获取活动工作簿每个Sheet名称
        SheetCount = ActiveWorkbook.Sheets.Count
    
        '重新定义数组元素数量
        ReDim SheetNames(1 To SheetCount)

        '记录原来的活动工作簿
        Set OldActive = ActiveSheet
   
        '用Sheet名称填充数组
        For i = 1 To SheetCount
            SheetNames(i) = ActiveWorkbook.Sheets(i).Name
        Next i
   
        '对数组进行排序
        Call BubbleSort(SheetNames)
   
        '关闭屏幕更新
        Application.ScreenUpdating = False
    
        '移动Sheet
        For i = 1 To SheetCount
            ActiveWorkbook.Sheets(SheetNames(i)).Move Before:=ActiveWorkbook.Sheets(i)
        Next i

        '恢复之前的活动工作表
        OldActive.Activate
    
    End Sub

    Sub BubbleSort(List() As String)
    '按照字母升序排序法
        Dim First As Long, Last As Long
        Dim i As Long, j As Long
        Dim Temp As String
        First = LBound(List)
        Last = UBound(List)
        For i = First To Last - 1
            For j = i + 1 To Last
                If UCase(List(i)) > UCase(List(j)) Then
                    Temp = List(j)
                    List(j) = List(i)
                    List(i) = Temp
                End If
            Next j
        Next i
    End Sub
    ```

## 补充
- 可以在宏最上方添加Option Explicit代码，声明所有变量都需要先定义才能使用，可以避免变量因名拼写等错误带来的结果错误。

- 为了使在其他的工作簿也能使用代码，可以将代码保存在“个人宏工作簿”，即在工程窗口中的Personal.xlsb中编辑代码。
执行宏的方法：
   - 按Alt+F8打开宏对话框，选择宏；
   - 对宏设置快捷键，通过快捷键打开；
   - 将宏添加到功能区。

- 该实例严格来讲并不符合逻辑，例如Sheet10会排在Sheet2之前。