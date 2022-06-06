## Find the largest width and the largest height from all fields in a group of documents
this runs in a script locator on any document. It works in KTA because it loops through all the xdocs in the same folder as the xdoc being tested.

```
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
