'-----------------------------------------------------------------------------------------------------------------------------------------------------
' Projekt      :  CTRL_Task
' Version      :  1.0
'-----------------------------------------------------------------------------------------------------------------------------------------------------
' Datum        :  11.03.2009  #WU   Erstellt
' Geprueft     :  -
' Geaendert    :  -
' Softwaredoku :  -
' Online Help  :  -
' Bemerkungen  :  Fertig
'-----------------------------------------------------------------------------------------------------------------------------------------------------
' Klassen      :  cls_Main
'-----------------------------------------------------------------------------------------------------------------------------------------------------
' Beschreibung :  - Verwaltet den Applikation - Start mit der WindowsFormsApplicationBase Klasse.
'--------------------------------------------------------------------------------------------+--------------------------------------------------------

Option Strict On
Option Explicit On

''' <summary>
''' Verwaltet den Applikationsstart mit der WindowsFormsApplicationBase Klasse.
''' </summary>
''' <remarks>Die Sub Main ist shared (static).</remarks>
Public Class cls_Main

#Region "Strukturen"

#End Region

#Region "Variablen, Instanzen"

   Public Shared clsWinApp As cls_WinApp                                                     ' Windows Applikation Instanz deklarieren

#End Region

#Region "Properties"

#End Region

#Region "Ereignisse"

#End Region

#Region "Methoden, Funktionen"

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Methode main
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Initialisiert die Windows Applikation Instanz.
   ''' </summary>
   ''' <remarks></remarks>
   <System.STAThread()> _
   Public Shared Sub Main(ByVal Args() As String)
      clsWinApp = cls_WinApp.GetInstance                                                     ' Windows Applikation Instanz erstellen

      clsWinApp.Run(Args)                                                                    ' Windows Applikation starten

      If clsWinApp IsNot Nothing Then
         clsWinApp.Dispose()
         clsWinApp = Nothing                                                                 ' Windows Applikation Instanz freigeben
            Try
                System.Diagnostics.EventLog.WriteEntry( _
                                        System.AppDomain.CurrentDomain.FriendlyName, _
                                        System.Environment.NewLine + "CTRL.Net_Task finish", _
                                        System.Diagnostics.EventLogEntryType.Information)           ' Eintrag in Windows Event erstellen
            Catch ex As System.Exception                                                           ' Exception auswerten
            End Try

        End If



   End Sub

#End Region

End Class
