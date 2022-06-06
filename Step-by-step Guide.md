# Step by Step Guide.
This guide will show you how to run the Threshold Optimizer. You will first need to have a project that can run the **Extraction Benchmark** successfully.  
See [Anonymizing Zones from Documents.md](Anonymizing%20Zones%20from%20Documents.md) for an example of how to configure the Extraction Benchmark.  
![image](https://user-images.githubusercontent.com/103566874/172147968-30214ab7-84df-434b-b434-a53b3ada4247.png)  

## Configure the Threshold Optimizer
1. Add the following script to the  Project level script.
*You will also need to add the functions [String_LevenshteinDistance/Min/Max](https://github.com/KofaxTransformation/KTScripts/blob/5f0141f7a1aac5d439f46074b54df956827f1cb8/FuzzyMatch.vb#L24) and [FileName_WithoutPath](https://github.com/KofaxTransformation/KTScripts/blob/a08c90037fc4d0cc200722557c3ecaab27a2ab4a/File%20System.vb#L80)  
The event  **Document_AfterProcess** is called after the locators have run and **after** the fields have been both formatted and validated - this is important because we are looking at the **valid** value on each field.*
```vb
'#Language "WWB-COM"
Option Explicit
' Class script: Document

Private Sub Document_AfterProcess(ByVal pXDoc As CASCADELib.CscXDocument)
   Dim F As Long, Field As CscXDocField, TruthDoc As New CscXDocument, Truth As CscXDocField
   TruthDoc.Load(pXDoc.FileName)
   Open "c:\temp\parascript_alpha.txt" For Append As #1
   For F=0 To pXDoc.Fields.Count-1
      Set Field=pXDoc.Fields(F)
      If Field.PageIndex>-1 And TruthDoc.Fields.Exists(Field.Name) Then
         Set Truth=TruthDoc.Fields.ItemByName(Field.Name)
         If Truth.Text<>"" Then
            Print #1, FileName_WithoutPath(pXDoc.FileName)   & vbTab & vbTab & Field.Name & vbTab & Truth.Text & vbTab & Field.Text;
            Print #1, vbTab;
            Print #1, Format(Field.Confidence,"0.00%") & vbTab & Format(String_LevenshteinDistance(Field.Text,Truth.Text,IgnoreCase:=True))
         End If
      End If
   Next
   Close #1
End SubPrivate Sub Document_AfterProcess(ByVal pXDoc As CASCADELib.CscXDocument)
   Dim F As Long, Field As CscXDocField, TruthDoc As New CscXDocument, Truth As CscXDocField
   TruthDoc.Load(pXDoc.FileName)
   Open "c:\temp\parascript_alpha.txt" For Append As #1
   For F=0 To pXDoc.Fields.Count-1
      Set Field=pXDoc.Fields(F)
      If Field.PageIndex>-1 And TruthDoc.Fields.Exists(Field.Name) Then
         Set Truth=TruthDoc.Fields.ItemByName(Field.Name)
         If Truth.Text<>"" Then
            Print #1, FileName_WithoutPath(pXDoc.FileName)   & vbTab & vbTab & Field.Name & vbTab & Truth.Text & vbTab & Field.Text;
            Print #1, vbTab;
            Print #1, Format(Field.Confidence,"0.00%") & vbTab & Format(String_LevenshteinDistance(Field.Text,Truth.Text,IgnoreCase:=True))
         End If
      End If
   Next
   Close #1
End Sub
```
2. Configure your locators and profiles however you like. My recommendation is to only change one value at a time and then run the Optimizer - in this way you will see what every checkbox, radio button and value has on your project. 
3. Select all of your documents (CTRL-A) in the Test Window.
4. Go to **Visual Studio Code** and delete the contents of file (CTRL-A, Delete) and then save it (CTRL-S).
5. Run "Extact Docmuents" (F6) to test all of your documents.
*while the documents are being extracted you can view the live updates results file in **Visual Studio Code** l
## Copy data to Excel
1. When Extraction has finished, copy the data from the output file (CTRL-A, CTRL-C).
2. Duplicate the Worksheet in the Excel Document. Rename it. Paste data into A7 (CTRL-V).
3. Add your data to the Summary page by just changing the reference of the cell in column A to point to the first cell in your new Worksheet.




