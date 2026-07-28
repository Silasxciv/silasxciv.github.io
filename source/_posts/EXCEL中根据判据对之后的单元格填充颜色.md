---
layout: post
title: EXCEL中根据判据对之后的单元格填充颜色
comments: true
tags:
  - EXCEL
categories: 
  - 教程
date: 2025-03-25 08:22:13
---
EXCEL中G55到G44单元格填有判据，I5到CN44单元格填有数据，根据判据，对数据单元格进行颜色填充。
<!--more-->
```
Sub 根据判据自动填充颜色()
    Application.ScreenUpdating = False
    Application.Calculation = xlCalculationManual
    
    Dim ws As Worksheet
    Dim dataCell As Range
    Dim criteria As String
    Dim rowNum As Long, colNum As Long
    Dim val As Double, tolerance As Double
    Dim maxVal As Double, minVal As Double
    Dim rgbGreen As Long, rgbRed As Long
    
    ' 设置颜色值
    rgbGreen = RGB(144, 238, 144)   ' 浅绿色
    rgbRed = RGB(255, 182, 193)     ' 浅红色
    
    ' 设置工作表和范围
    Set ws = ActiveSheet
    Set rngData = ws.Range("I5:CN44")     ' 数据区域
    
    ' 清除原有颜色
    rngData.Interior.ColorIndex = xlNone
    
    ' 遍历每一行 (5到44行)
    For rowNum = 5 To 44
        ' 检查对应的判据行是否存在 (G5对应I5行，G6对应I6行...)
        If rowNum <= 44 Then
            criteria = Trim(ws.Cells(rowNum, "G").Text)
            
            ' 只处理非空判据
            If criteria <> "" Then
                ' 情况1: 简单数值 (如19.50)
                If IsNumeric(criteria) Then
                    val = CDbl(criteria)
                    
                    ' 遍历I到CN列 (9到98列)
                    For colNum = 9 To 98
                        Set dataCell = ws.Cells(rowNum, colNum)
                        
                        ' 只处理有数据的单元格
                        If Not IsEmpty(dataCell) And IsNumeric(dataCell.Value) Then
                            If CDbl(dataCell.Value) = val Then
                                dataCell.Interior.Color = rgbGreen
                            Else
                                dataCell.Interior.Color = rgbRed
                            End If
                        End If
                    Next colNum
                
                ' 情况2: 带公差的值 (如-25±3)
                ElseIf InStr(criteria, "±") > 0 Then
                    Dim parts() As String
                    parts = Split(criteria, "±")
                    
                    If UBound(parts) = 1 Then
                        If IsNumeric(Trim(parts(0))) And IsNumeric(Trim(parts(1))) Then
                            val = CDbl(Trim(parts(0)))
                            tolerance = CDbl(Trim(parts(1)))
                            minVal = val - tolerance
                            maxVal = val + tolerance
                            
                            For colNum = 9 To 98
                                Set dataCell = ws.Cells(rowNum, colNum)
                                
                                If Not IsEmpty(dataCell) And IsNumeric(dataCell.Value) Then
                                    If CDbl(dataCell.Value) >= minVal And CDbl(dataCell.Value) <= maxVal Then
                                        dataCell.Interior.Color = rgbGreen
                                    Else
                                        dataCell.Interior.Color = rgbRed
                                    End If
                                End If
                            Next colNum
                        End If
                    End If
                
                ' 情况3: 不等式 (如≤13)
                ElseIf Left(criteria, 1) = "≤" Or Left(criteria, 1) = "<" Then
                    If IsNumeric(Mid(criteria, 2)) Then
                        maxVal = CDbl(Mid(criteria, 2))
                        
                        For colNum = 9 To 98
                            Set dataCell = ws.Cells(rowNum, colNum)
                            
                            If Not IsEmpty(dataCell) And IsNumeric(dataCell.Value) Then
                                If CDbl(dataCell.Value) <= maxVal Then
                                    dataCell.Interior.Color = rgbGreen
                                Else
                                    dataCell.Interior.Color = rgbRed
                                End If
                            End If
                        Next colNum
                    End If
                End If
                ' 其他格式的判据不做处理
            End If
        End If
    Next rowNum
    
    Application.ScreenUpdating = True
    Application.Calculation = xlCalculationAutomatic
    MsgBox "颜色填充完成!", vbInformation
End Sub
```