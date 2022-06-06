# Anonymizing Zones
With the steps below I copied zones out of sensitive documents into their own documents and deleted the more unusual names. This leaves me with documents only containing first or last names - the documents are sorted alphabetically so the link between first anem and last name is broken.

## Find the largest width and the largest height from all fields in a group of documents
My documents have zones of all different sizes and locations all over the documents. I manually created the zones. Now I want to chop all of these zones out of the documents and make new documents all of exactly the same size with a zone size that is big enough to read ALL of these zones.  
This runs in a script locator on any document. It works in KTA Transformation Designer by pressing F7 on any document, because it loops through all the xdocs in the same folder as the xdoc being tested.  
In my case, it found the widest field was 1963 pixels and the heighest field was 436 pixels.
```vb
Private Sub SL_Dim_LocateAlternatives(ByVal pXDoc As CASCADELib.CscXDocument, ByVal pLocator As CASCADELib.CscXDocField)
   Dim FileName As String, Field As CscXDocField, Path As String, F As Long, Truth As New CscXDocument, I As Long, Alt As CscXDocFieldAlternative
   Dim W As Long, H As Long
   Path = Left (pXDoc.FileName, InStrRev(pXDoc.FileName,"\"))
   ChDir Path
   FileName = Dir("*.xdc")
    While FileName <> ""
      Truth.Load(FileName) ' Load the XDoc from the file system.
      For F=0 To Truth.Fields.Count-1
         Set Field=Truth.Fields(F)
         If Field.PageIndex>-1 And Field.Text<>"" Then
            If Field.Width>W Then W=Field.Width
            If Field.Height>H Then H=Field.Height
         End If
      Next
      FileName = Dir()
    Wend
   Set Alt= pLocator.Alternatives.Create
   Alt.Confidence=1
   With Alt.SubFields.Create("W")
      .Text=CStr(W)
      .Confidence=1
   End With
   With Alt.SubFields.Create("H")
      .Text=CStr(H)
      .Confidence=1
   End With
End Sub
```
## Export all Fields as their own TIF files with the file name as the field value and every TIF image EXACTLY the same size.
each document is now exactly one zone with width =1963 and height = 436. Each zone is padded 20 pixels on all four size.  
This means one single AZL with size (1963,436) can read them all perfectly.  
I can now benchmark the OCR engine
```vb
' Class script: Document

Private Sub SL_Dim_LocateAlternatives(ByVal pXDoc As CASCADELib.CscXDocument, ByVal pLocator As CASCADELib.CscXDocField)
   Dim FileName As String, Field As CscXDocField, Path As String, F As Long, Truth As New CscXDocument, I As Long, Alt As CscXDocFieldAlternative
   Dim W As Long, H As Long
   Randomize 'Seed the random number generator with the current time
   Path = Left (pXDoc.FileName, InStrRev(pXDoc.FileName,"\"))
   ChDir Path
   FileName = Dir("*.xdc")
    While FileName <> ""
      Truth.Load(FileName) ' Load the XDoc from the file system.
      For F=0 To Truth.Fields.Count-1
         Set Field=Truth.Fields(F)
         If Field.PageIndex>-1 And Field.Text<>"" Then
            Select Case Field.Name
            Case "FirstName", "LastName"
               Document_ExportField(Truth, F, 1963,436, 20, "C:\temp\out")
            End Select
         End If
      Next
      FileName = Dir()
    Wend
   Set Alt= pLocator.Alternatives.Create
   Alt.Confidence=1
   With Alt.SubFields.Create("W")
      .Text=CStr(W)
      .Confidence=1
   End With
   With Alt.SubFields.Create("H")
      .Text=CStr(H)
      .Confidence=1
   End With
End Sub

Private Sub Document_ExportField(ByVal XDoc As CASCADELib.CscXDocument, FieldId As Long, Width As Long, Height As Long, Padding As Long, Path As String)
   'Create a tif image for every field. Each tif image has exactly the same width and height and same padding around the zone.
   'This makes it easy For an AZL To Read All of them
   Dim FileName As String, Field As CscXDocField, F As Long, I As Long, Image As CscImage
   Set Field=XDoc.Fields(F)
   If InStr(Field.Text, "/") >0 Then Exit Sub ' File name would have an illegal "/" in it, so skip.
   If Field.PageIndex>-1 And Field.Text<>"" Then ' this field has coordinates and text
      Set Image=New CscImage
      Image.CreateImage(CscImgColFormatBinary,Width+Padding*2,Height+Padding*2,Image.XResolution,Image.YResolution) ' Make a Black&White image
      Image.CopyRect(XDoc.CDoc.Pages(Field.PageIndex).GetImage.BinarizeWithVRS(),Field.Left,Field.Top,Padding,Padding,Field.Width,Field.Height)
      I=0
      Do 'increment I if the file already exists
         I=I+1
         FileName=Path & "\" & Field.Text & "_" & Format(I,"00") & ".tif"
      Loop While File_Exists (FileName)
      Image.Save(FileName)
   End If
End Sub

Function File_Exists(file As String) As Boolean
   On Error GoTo ErrorHandler
   Return (GetAttr(file) And vbDirectory) = 0
   Exit Function
   ErrorHandler:
End Function
```
# Put the truth back into the XDocuments
I added an Advanced Zone Locator (AZL) to the project. Gave it the first image as the sample, set the registration to **None**  and added one Text Zone that covered the entire of the document  
![image](https://user-images.githubusercontent.com/103566874/172140066-6382494c-9623-4656-abf3-b5ed4b4fb7dc.png)  
I then extracted all of the documents (CTRL-A, F7)  
I added the **Name** field the the **Details**  
![image](https://user-images.githubusercontent.com/103566874/172140552-8c9ed306-9ca4-4a5c-9187-b53afc19ba28.png)  
and can now see all the OCR results with the documents  
![image](https://user-images.githubusercontent.com/103566874/172140961-5a0c4219-68ff-46d6-b0ab-83ddbfcbb433.png)  

Here you can see a wrong OCR result with confidence of 54% in the **Extraction Results** Window.  
![image](https://user-images.githubusercontent.com/103566874/172140798-406dd3e2-8589-45bd-a89f-cc3a13174f45.png)  
These results contain errors, but we can see the correct text in the file name.
The following script in the document class will replace the OCR text with the correct text from the filename and set the confidence to 100%.  

```vb
Private Sub Document_AfterExtract(ByVal pXDoc As CASCADELib.CscXDocument)
   Dim Filename As String, Field As CscXDocField
   Set Field=pXDoc.Fields.ItemByName("Name")
   Filename = Mid(pXDoc.FileName,InStrRev(pXDoc.FileName,"\")+1) ' the filename is everything after the last backslash
   Field.Text=UCase(Left(Filename,InStr(Filename,"_")-1))  ' True field value is everything left of _ in the file name
   Field.Confidence=1.00 ' confidence = 100%
   Field.ExtractionConfident=True 'Make the green check mark in the Extraction Results Window
End Sub
```
and re-extracted everything (CTRL-A, F7) and now every document is perfect!  
![image](https://user-images.githubusercontent.com/103566874/172145235-870fd862-ff1b-4895-865b-952e13a2e400.png)  
Note the green check mark **ExtractionConfident=true** and confidence =100%  
![image](https://user-images.githubusercontent.com/103566874/172145297-c6c9fce5-14fe-4e0b-af90-da741a1d5ea1.png)  
I saved all the documents   
![image](https://user-images.githubusercontent.com/103566874/172145368-61d6e9f1-99cf-417b-bf76-dde505f70dd0.png)  
and made a backup of the folder containing this perfect truth (Because it is so easily to mess this up and lose it all!)  
![image](https://user-images.githubusercontent.com/103566874/172142510-4caf7fb2-0de5-4ac0-8451-1751b28ebf90.png)

remove this script before continuing, otherwise the benchmark will look perfect 😊.  


# Run a normal Extraction Benchmark
I Right-clicked on the test set and selected **Use as Benchmark Set**.  
![image](https://user-images.githubusercontent.com/103566874/172143077-287da2ca-d518-496c-918e-08d3b0886da8.png)  
![image](https://user-images.githubusercontent.com/103566874/172143142-1196fa41-9a74-418c-af66-20c491c853aa.png)  
I have 425 documents and the benchmark takes longer than 1 minute to run, so I made a tiny test set so that I can test that the benchmark is working correctly becfore I run the full thing!  I created a **Document Subset** called **small**.  
![image](https://user-images.githubusercontent.com/103566874/172145875-77501a33-2db3-4dfe-99c0-4df71fc0a13e.png)  
I dragged a few documents into the **small** subset  
![image](https://user-images.githubusercontent.com/103566874/172146644-ff7a0ab9-96f0-4e6e-ba93-e969269d13c0.png)  
I set the **small** subset to be the **Default Document Subset**  
![image](https://user-images.githubusercontent.com/103566874/172146722-df7b5832-a1a4-491e-ae95-a33cc19de765.png)  
and saved the Benchmark Set.  
![image](https://user-images.githubusercontent.com/103566874/172146880-9d55a6c1-1b23-418d-bc1d-90bb70eb14ee.png)  

I ran **Extraction Benchmark** from the **Process** Menu on this tiny set of 6 documents.  
![image](https://user-images.githubusercontent.com/103566874/172143454-7ac5d4c2-8a68-407c-a7b3-f9bbd73e1ea5.png)  
In the results I see that I don't have optimal results. I would like to know what the best choice is for the confidence threshold, which defaulted to 80%.  

Now I am ready to run the [Threshold Optimizer](readme.md) to find the best threshold value!

 








