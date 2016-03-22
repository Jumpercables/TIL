
## Toggle (On/Off) ArcFM Auto Updater Framework
The snippet can be used to disable or enable the ArcFM Auto Updater Framework.

```vba
' Reference Miner & Miner 9.x Geodatabase Library

Public Sub ToggleAU()

    Dim pAU As IMMAutoUpdater
    Set pAU = New MMAutoUpdater

    If pAU.AutoUpdaterMode = mmAUMNoEvents Then
        pAU.AutoUpdaterMode = mmAUMArcMap
    Else
        pAU.AutoUpdaterMode = mmAUMNoEvents
    End If    

End Sub
```

## ArcFM Attribute Editor
The snippet can be used to show the ArcFM Attribute Editor.
```vba
' Reference Miner & Miner 9.x Desktop Library

Public Sub ShowAttributeEditor()

Dim pEditor As IEditor
Set pEditor = Application.FindExtensionByName("ESRI Object Editor")

Dim pUID As New UID
pUID.Value = "{B2575F11-BD46-11D3-BD53-00500462EE0B}"

Dim pAttributeEditor As IMMAttributeEditor
Set pAttributeEditor = pEditor.FindExtension(pUID)

If Not IsNull(pAttributeEditor) Then
    pAttributeEditor.Show 0
End If

End Sub
```
- The `IMMAttributeEditor` has been marked to be deprecated at a later date.

## Non Locking Reconcile
The snippet will reconcile the version with the ArcFM Auto Updater Framework *disabled*.

```vba

' Reference Miner & Miner 9.x Geodatabase Library

Public Sub NonLockingReconcile()

On Error GoTo ErrorHandler

  Dim conflictWindow As IConflictsWindow3
  Dim editor As IEditor
  Dim editorID As New UID
  Dim conflictID As New UID
  Dim hasConflict As Boolean
  Dim workspace As IWorkspace
  Dim version As IVersion
  Dim versionEdit As IVersionEdit4
  Dim versionInfo As IVersionInfo

  conflictID = "esriEditor.ConflictsWindow"
  editorID = "esriEditor.Editor"

  Set editor = Application.FindExtensionByCLSID(editorID)
  Set conflictWindow = editor.FindExtension(conflictID)

  If (editor.EditState <> esriStateEditing) Then
    MsgBox "You must be editing to reconcile."
    Exit Sub
  End If

  Dim au As IMMAutoUpdater
  Set au = New MMAutoUpdater
  au.AutoUpdaterMode = mmAUMNoEvents

  Set workspace = editor.EditWorkspace
  Set version = workspace
  Set versionEdit = workspace
  Set versionInfo = version.versionInfo

  hasConflict = versionEdit.Reconcile4(versionInfo.Parent.VersionName, False, False, True, True)

  If (hasConflict) Then
    conflictWindow.Visible = True
    conflictWindow.Reset
  Else
   MsgBox "Reconcile completed."
  End If

  Exit Sub

ErrorHandler:
    MsgBox Err.Description
End Sub
```
