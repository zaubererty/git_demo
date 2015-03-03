'-----------------------------------------------------------------------------------------------------------------------------------------------------
' Projekt      :  APP Task
' Version      :  2.0
'-----------------------------------------------------------------------------------------------------------------------------------------------------
' Datum        :  19.04.2008  #PO   Erstellt
' Geprueft     :  23.04.2008  #PO
' Geaendert    :  -
' Softwaredoku :  -
' Online Help  :  -
' Bemerkungen  :  Fertig
'-----------------------------------------------------------------------------------------------------------------------------------------------------
' Form         :  frm_Main
'-----------------------------------------------------------------------------------------------------------------------------------------------------
' Beschreibung :  Zum Anzeigen der Applikationsformen und des Infofensters.
'--------------------------------------------------------------------------------------------+--------------------------------------------------------

Option Strict On
Option Explicit On

''' <summary>
''' Haupt Form. Ermoeglicht das Anzeigen der Applikationsformen.
''' </summary>
''' <remarks>Diese Form ist immer erreichbar (sichtbar).</remarks>
Public Class frm_Main

#Region "Strukturen"

#End Region

#Region "Variablen, Instanzen"

   Private _blnCtrlForm As Boolean                                                            ' Form APP Status
   Private _blnDebug_Active As Boolean                                                       ' Form Debug Status
   Private _strPassword As String                                                            ' Momentanes Passwort

   Private _clsWinAPP As cls_WinApp                                                          ' Windows Applikations Klasse
   Private _objExcReturn As System.Windows.Forms.DialogResult                                ' Dialog Return Objekt
   Private _blnAppExit As Boolean                                                            ' Flag Applikation wird beendet
   Private _objButton As System.Windows.Forms.Button                                         ' Button Objekt deklarieren
   Private WithEvents _objClock As System.Windows.Forms.Timer                                ' Uhr Timer deklarieren

   ' Delegaten fuer Invoke Methoden
   Public Delegate Sub OnApplicationExit()                                                   ' OnApplicationExit Delegat
   Public Delegate Sub OnToggleVisibleAPPForm()                                              ' OnToggleVisibleAPPForm Delegat

#End Region

#Region "Properties"

#End Region

#Region "Ereignis Sender, Receiver"

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis OnThreadEnd (Sender)
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Wird bei Thread Ende (Form) gesendet.
   ''' </summary>
   ''' <param name="Sender">Eigene Form.</param>
   ''' <remarks></remarks>
   Public Event OnThreadEnd(ByVal Sender As Object)

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis ApplicationExit (Receiver)
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Wird beim Beenden der Applikation empfangen.
   ''' </summary>
   ''' <param name="sender">Objekt, welches gesendet hat.</param>
   ''' <param name="e">Ereignis Argumente.</param>
   ''' <remarks></remarks>
   Private Sub ApplicationExit(ByVal sender As Object, ByVal e As System.Exception)
      Dim objDelegate As New OnApplicationExit(AddressOf Inv_Formclose)                      ' Delegate Objekt erstellen

      _blnAppExit = True                                                                     ' Flag Applikation wird beendet setzen
      If Me.Created Then                                                                     ' Wenn Form erstellt, dann
         Invoke(objDelegate)                                                                 ' Invoke aufrufen
      End If
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis ApplicationExit (Receiver) Invoke Methode
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Wird beim Beenden der Applikation empfangen, schliesst die Form.
   ''' </summary>
   ''' <remarks></remarks>
   Private Sub Inv_Formclose()
      Me.Close()                                                                             ' Schliesst die Form 
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Methode (Ereignis) Toggle_VisibleAPPForm (aus IIS_Task)
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Zeigt oder versteckt die APP Form.
   ''' </summary>
   ''' <remarks></remarks>
   Public Sub Toggle_VisibleAPPForm()
      Dim objDelegate As New OnToggleVisibleAPPForm(AddressOf Inv_Toggle_VisibleAPPForm)     ' Delegate Objekt erstellen

      If Me.Created Then                                                                     ' Wenn Form erstellt, dann
         Invoke(objDelegate)                                                                 ' Invoke aufrufen
      End If
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Methode (Ereignis) Toggle_VisibleAPPForm (aus IIS_Task) Invoke Methode
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Zeigt oder versteckt die APP Form.
   ''' </summary>
   ''' <remarks></remarks>
   Private Sub Inv_Toggle_VisibleAPPForm()
      Dim e As System.EventArgs = New System.EventArgs                                       ' Event Argumente
      Dim es As System.Windows.Forms.KeyEventArgs = New  _
                        System.Windows.Forms.KeyEventArgs(System.Windows.Forms.Keys.Enter)   ' Key Event Argumente
      Dim blnCtrlForm As Boolean = _blnCtrlForm                                                ' Form Status zwischenspeichern

      Me.btn_Click(Me.btnFormCtrl, e)                                                         ' btnFormAPP clicken

      If blnCtrlForm = False Then                                                             ' Wenn APPForm nicht angezeigt war, dann
         Me.txtPassword.Text = Me.GeneratePassword                                           ' Passwort in Textbox ausgeben und
         Me.txtPassword_KeyDown(Me, es)                                                      ' txtPassword_KeyDown ausloesen
      End If
   End Sub


   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis OnShowAnlageForm (Sender)
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Wird gesendet, wenn die Form der Anlage angezeigt werden soll. 
   ''' </summary>
   ''' <remarks>Der Event wird an die cls_WinAPP weitergeleitet</remarks>
   Public Event OnShowAnlageForm(ByVal aVisible As Boolean)

#End Region

#Region "Methoden, Funktionen, Form Ereignisse"

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Methode New (Konstruktor)
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Initialisiert die Komponenten und Daten.
   ''' </summary>
   ''' <remarks></remarks>
   Public Sub New()
      InitializeComponent()                                                                  ' This call is required by the Windows Form Designer

      _clsWinAPP = cls_WinApp.GetInstance                                                    ' Klasse Windows Applikation instanzieren
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis frm_Main_FormClosed
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Speichert die orm Location und gibt die Instanzen frei.
   ''' </summary>
   ''' <param name="sender">Objekt, welches gesendet hat.</param>
   ''' <param name="e">Ereignis Argumente.</param>
   ''' <remarks></remarks>
   Private Sub frm_Main_FormClosed(ByVal sender As Object, ByVal e As System.Windows.Forms.FormClosedEventArgs) Handles Me.FormClosed
      Try
         My.Settings_Main.Default.Save()                                                     ' Form Daten speichern
         If _clsWinAPP IsNot Nothing Then
            RemoveHandler _clsWinAPP.OnApplicationExit, AddressOf Me.ApplicationExit         ' OnApplikationExit Handler entfernen
            _clsWinAPP.Dispose()
            _clsWinAPP = Nothing                                                             ' Klasse _clsWinAPP aufraeumen
         End If
         '         RaiseEvent OnThreadEnd(Me)                                                          ' Thread Ende signalisieren
      Catch ex As System.Exception                                                           ' Exception auswerten
         _objExcReturn = _clsWinAPP.clsException.HandledException(ex, , _
                                          System.Windows.Forms.MessageBoxIcon.Information, _
                                          System.Windows.Forms.MessageBoxButtons.OK)
      Finally                                                                                ' Instanzen freigeben
         If _objClock IsNot Nothing Then
            _objClock.Stop()
            _objClock.Dispose()
            _objClock = Nothing
         End If
      End Try
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis frm_Main_FormClosing
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Beendet die Applikation.
   ''' </summary>
   ''' <param name="sender">Objekt, welches gesendet hat.</param>
   ''' <param name="e">Ereignis Argumente.</param>
   ''' <remarks>Im Run Mode wird der Benutzer noch gefragt, ob er die Applikation beenden will.</remarks>
   Private Sub frm_Main_FormClosingOrg(ByVal sender As Object, ByVal e As System.Windows.Forms.FormClosingEventArgs) Handles Me.FormClosing
      Try
         If System.Diagnostics.Debugger.IsAttached = False Then                              ' Wenn Debugger nicht angebunden, dann
            If (e.CloseReason = System.Windows.Forms.CloseReason.UserClosing) And _
                                                               (_blnAppExit = False) Then    ' Wenn der Benutzer die Form schliesst, dann
               Dim objAssembly As cls_ProcFunc.Assembly = New cls_ProcFunc.Assembly(True)    ' Assembly Informationen ermitteln
               If System.Windows.Forms.MessageBox.Show( _
                     String.Format(My.Resources.Resources_Main.Messagebox_Close, _
                     objAssembly.AssemblyName), _
                     Me.Text, System.Windows.Forms.MessageBoxButtons.YesNo, _
                     System.Windows.Forms.MessageBoxIcon.Question, _
                     System.Windows.Forms.MessageBoxDefaultButton.Button2, _
                     System.Windows.Forms.MessageBoxOptions.DefaultDesktopOnly) = _
                                                System.Windows.Forms.DialogResult.No Then    ' Wenn Benutzer abbricht, dann
                  e.Cancel = True                                                            ' Event Argument Cancel setzen
               End If
            End If
         End If

         If e.Cancel = False Then                                                            ' Wenn kein Cancel, dann
            _clsWinAPP.blnNoMoreInvokesPlease = True
            ' Events nochmals ausfuehren, ansonsten wird beim Beenden der Applikation jeweils eine Invoke Exception geworfen
            System.Windows.Forms.Application.DoEvents()
            RaiseEvent OnThreadEnd(Me)                                                          ' Thread Ende signalisieren

            _blnAppExit = True                                                               ' Flag Applikation wird beendet setzen
         End If
      Catch ex As System.Exception                                                           ' Exception auswerten
         _objExcReturn = _clsWinAPP.clsException.HandledException(ex, , _
                                         System.Windows.Forms.MessageBoxIcon.Information, _
                                         System.Windows.Forms.MessageBoxButtons.OK)
      End Try
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis frm_Main_Load
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Initialisiert beim Laden die Form.
   ''' </summary>
   ''' <param name="sender">Objekt, welches gesendet hat.</param>
   ''' <param name="e">Ereignis Argumente.</param>
   ''' <remarks></remarks>
   Private Sub frm_Main_Load(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.Load
      Try
         Dim objAssembly As cls_ProcFunc.Assembly = New cls_ProcFunc.Assembly(True)          ' Assembly Informationen ermitteln
         Me.Text = objAssembly.AssemblyName + "-V" + objAssembly.AssemblyVersion      ' Form Name inkl. Hauptversion ausgeben

         Me.Location = My.Settings_Main.Default.frm_Main_Location                            ' Form Position setzen
         ShowClock()                                                                         ' Datum und Zeit ausgeben

         _objClock = New System.Windows.Forms.Timer                                          ' Uhr Timer instanzieren
         _objClock.Interval = 1000                                                           ' Uhr Timer Interval 1 Sekunde
         _objClock.Start()                                                                   ' Uhr Timer starten

         If _clsWinAPP.udtApp.bytAppForm = 1 Then                                            ' APP Button und Status initialisieren
            btnFormCtrl.Text = My.Resources.Resources_Main.Button_APP_Hide
            _blnCtrlForm = True
         Else
            btnFormCtrl.Text = My.Resources.Resources_Main.Button_APP_Show
         End If

         If (_clsWinAPP.udtApp.bytDebug_Active = 1) Then                                     ' Debug Button und Status initialisieren
            btnFormDebug.Text = My.Resources.Resources_Main.Button_Debug_Hide
            _blnDebug_Active = True
         Else
            btnFormDebug.Text = My.Resources.Resources_Main.Button_Debug_Show
         End If

         grpPassword.Text = My.Resources.Resources_Main.Password_Groupbox                    ' Groupbox beschriften

         AddHandler _clsWinAPP.OnApplicationExit, AddressOf Me.ApplicationExit               ' OnApplikationExit Handler einbinden
      Catch ex As System.Exception                                                           ' Exception auswerten
         _objExcReturn = _clsWinAPP.clsException.HandledException(ex, , _
                                         System.Windows.Forms.MessageBoxIcon.Warning, _
                                         System.Windows.Forms.MessageBoxButtons.OK)
      End Try
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis frm_Main_LocationChanged
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Ermittelt die Position beim Verschieben der Form.
   ''' </summary>
   ''' <param name="sender">Objekt, welches gesendet hat.</param>
   ''' <param name="e">Ereignis Argumente.</param>
   ''' <remarks>
   ''' Position wird nur im Status 'WindowState=normal' und 'Form Created' ermittelt.
   ''' Wichtig: Das LocationChanged Ereignis wird noch bevor die Form erstellt wurde (Load) aufgerufen!
   ''' </remarks>
   Private Sub frm_Main_LocationChanged(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.LocationChanged
      Try
         If ((Me.WindowState = System.Windows.Forms.FormWindowState.Normal) And _
                                                                  (Me.Created = True)) Then  ' Wenn Form Status 'normal', dann
            My.Settings_Main.Default.frm_Main_Location = Me.Location                         ' Form Position ablegen
         End If
      Catch ex As System.Exception                                                           ' Exception auswerten
         _objExcReturn = _clsWinAPP.clsException.HandledException(ex, , _
                                         System.Windows.Forms.MessageBoxIcon.Information, _
                                         System.Windows.Forms.MessageBoxButtons.OK)
      End Try
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis btn_Click
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Zeigt/versteckt die Applikationsformen.
   ''' </summary>
   ''' <param name="sender">Objekt, welches gesendet hat.</param>
   ''' <param name="e">Ereignis Argumente.</param>
   ''' <remarks></remarks>
   Private Sub btn_Click(ByVal sender As System.Object, ByVal e As System.EventArgs) Handles btnFormCtrl.Click, btnFormDebug.Click
      Try
         _objButton = DirectCast(sender, System.Windows.Forms.Button)                        ' Sendendes Button Objekt ermitteln
         If ((_objButton.Equals(btnFormCtrl) = True) And (_blnCtrlForm = True)) Then         ' Wenn CTRL Button und CTRL Form angezeigt, dann
            '            _clsWinAPP.frmMainCtrl.Visible = False                                                ' Form CTRL verstecken
            '           _clsWinAPP.frmMainCtrl.ShowInTaskbar = False                                          ' Form APP nicht im Task Manager anzeigen
            btnFormCtrl.Text = My.Resources.Resources_Main.Button_APP_Show                    ' Button Beschriftung aendern
            _blnCtrlForm = False                                                              ' Form APP wird nicht angezeigt
            RaiseEvent OnShowAnlageForm(False)
         Else
            Dim objButton As System.Windows.Forms.Button                                     ' Button Objekt deklarieren 
            For Each objButton In Me.pnlButton.Controls                                      ' Alle Button inaktiv setzen
               objButton.Enabled = False
            Next
            txtPassword.Enabled = True                                                       ' Passwort Textbox aktivieren
            txtPassword.Font = New System.Drawing.Font(txtPassword.Font, _
                                                        System.Drawing.FontStyle.Regular)    ' Passwort Textbox Font nicht Fett setzen
            txtPassword.Focus()                                                              ' Passwort Textbox den Focus geben
         End If
      Catch ex As System.Exception                                                           ' Exception auswerten
         _objExcReturn = _clsWinAPP.clsException.HandledException(ex, , _
                                         System.Windows.Forms.MessageBoxIcon.Warning, _
                                         System.Windows.Forms.MessageBoxButtons.OK)
      End Try
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis txtPassword_KeyDown
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Ermittelt Passwort Identifikation und zeigt die gewuenschte Form an.
   ''' </summary>
   ''' <param name="sender">Objekt, welches gesendet hat.</param>
   ''' <param name="e">Ereignis Argumente.</param>
   ''' <remarks></remarks>
   Private Sub txtPassword_KeyDown(ByVal sender As Object, ByVal e As System.Windows.Forms.KeyEventArgs) Handles txtPassword.KeyDown
      Try
         If e.KeyData = System.Windows.Forms.Keys.Enter Then                                 ' Wenn Enter Taste gedrueckt, dann
            If txtPassword.Text = GeneratePassword() Then                                    ' Wenn Passwort ok, dann
               If _objButton.Equals(btnFormCtrl) Then
                  '                  _clsWinAPP.frmMainCtrl.Visible = True
                  '                 _clsWinAPP.frmMainCtrl.ShowInTaskbar = True                                     ' Form APP im Task Manager anzeigen
                  btnFormCtrl.Text = My.Resources.Resources_Main.Button_APP_Hide              ' Button Beschriftung aendern
                  _blnCtrlForm = True                                                         ' Form APP wird angezeigt

                  ' Event feuern fuer die Anzeige der Mainform
                  RaiseEvent OnShowAnlageForm(True)

               ElseIf _objButton.Equals(btnFormDebug) Then                                   ' Wenn Debug Button, dann
                  _clsWinAPP.frmDebug.Visible = True                                         ' Form Debug anzeigen
                  _clsWinAPP.frmDebug.ShowInTaskbar = True                                   ' Form Debug im Task Manager anzeigen
                  btnFormDebug.Text = My.Resources.Resources_Main.Button_Debug_Hide          ' Button Beschriftung aendern
                  _blnDebug_Active = True                                                    ' Form Debug wird angezeigt
               End If
               txtPassword_Exit(True, e)                                                     ' Passwort Textbox verlassen
            Else
               txtPassword.Font = New System.Drawing.Font(txtPassword.Font, _
                                                            System.Drawing.FontStyle.Bold)   ' Passwort Textbox Font Fett setzen
               txtPassword.BackColor = System.Drawing.Color.DarkRed                          ' Hintergrund Farbe Rot
               txtPassword.PasswordChar = Nothing                                            ' Keine Passwort Zeichen
               txtPassword.Text = My.Resources.Resources_Main.Password_Wrong                 ' Text in Textbox ausgeben
               txtPassword_Exit(False, e)                                                    ' Passwort Textbox verlassen
            End If
         ElseIf e.KeyData = System.Windows.Forms.Keys.Escape Then                            ' Wenn Escape Taste gedrueckt, dann
            txtPassword_Exit(True, e)                                                        ' Passwort Textbox verlassen
         End If
      Catch ex As System.Exception                                                           ' Exception auswerten
         _objExcReturn = _clsWinAPP.clsException.HandledException(ex, , _
                                         System.Windows.Forms.MessageBoxIcon.Warning, _
                                         System.Windows.Forms.MessageBoxButtons.OK)
      End Try
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Methode txtPassword_Exit
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Loescht die Passworteingabe und aktiviert die Buttons. 
   ''' </summary>
   ''' <param name="Clear">True=Passwort Textbox loeschen und Beep unterdruecken.</param>
   ''' <param name="e">KeyEvent Argumente.</param>
   ''' <remarks></remarks>
   Private Sub txtPassword_Exit(ByVal Clear As Boolean, ByVal e As System.Windows.Forms.KeyEventArgs)
      Try
         If Clear = True Then                                                                ' Wenn Passwort Textbox loeschen verlangt, dann
            txtPassword.Text = ""                                                            ' Passwort Textbox loeschen
            e.SuppressKeyPress = True                                                        ' Eingabe Zeichen nicht an Passwort Textbox weiterleiten;
         End If                                                                              ' unterdrueckt auch den Beep!

         txtPassword.Enabled = False                                                         ' Passwort Textbox deaktivieren

         Dim Button As System.Windows.Forms.Button                                           ' Button Objekt deklarieren  
         For Each Button In Me.pnlButton.Controls                                            ' Alle Button aktiv setzen
            Button.Enabled = True
         Next

         _objButton.Focus()                                                                  ' Dem gedrueckten Button den Focus geben
      Catch ex As System.Exception                                                           ' Exception auswerten
         _objExcReturn = _clsWinAPP.clsException.HandledException(ex, , _
                                         System.Windows.Forms.MessageBoxIcon.Warning, _
                                         System.Windows.Forms.MessageBoxButtons.OK)
      End Try
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis txtPassword_GotFocus
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Loescht die Passwort Textbox und setzt die Passwortzeichen.
   ''' </summary>
   ''' <param name="sender">Objekt, welches gesendet hat.</param>
   ''' <param name="e">Ereignis Argumente.</param>
   ''' <remarks></remarks>
   Private Sub txtPassword_GotFocus(ByVal sender As Object, ByVal e As System.EventArgs) Handles txtPassword.GotFocus
      Try
         txtPassword.Text = ""                                                               ' Passwort Textbox loeschen
         txtPassword.PasswordChar = System.Convert.ToChar("*")                               ' Passwortzeichen '*' setzen
         txtPassword.BackColor = System.Drawing.SystemColors.Window                          ' Hinergrundfarbe setzen
      Catch ex As System.Exception                                                           ' Exception auswerten
         _objExcReturn = _clsWinAPP.clsException.HandledException(ex, , _
                                         System.Windows.Forms.MessageBoxIcon.Information, _
                                         System.Windows.Forms.MessageBoxButtons.OK)
      End Try
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Funktion GeneratePassword
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Erzeugt Passwort aus Datum und Zeit (JJJJMMTTHH).
   ''' </summary>
   ''' <returns>Generiertes Passwort als String.</returns>
   ''' <remarks></remarks>
   Private Function GeneratePassword() As String
      Try
         Dim objSB As System.Text.StringBuilder = New System.Text.StringBuilder              ' Stringbuilder Objekt instanzieren
         Dim objDateTime As System.DateTime = System.DateTime.Now                            ' Datum und Zeit einlesen
         objSB.Append(objDateTime.Year.ToString("0000"))                                     ' Passwort zusammenstellen
         objSB.Append(objDateTime.Month.ToString("00"))
         objSB.Append(objDateTime.Day.ToString("00"))
         objSB.Append(objDateTime.Hour.ToString("00"))
         Return objSB.ToString                                                               ' Passwort zurueckgeben
      Catch ex As System.Exception                                                           ' Exception auswerten
         _objExcReturn = _clsWinAPP.clsException.HandledException(ex, , _
                                         System.Windows.Forms.MessageBoxIcon.Information, _
                                         System.Windows.Forms.MessageBoxButtons.OK)
         Return ""                                                                           ' Leerstring zurueckgeben
      End Try
   End Function

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Ereignis Clock_Tick
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Gibt das Datum und die Zeit im Sekundentakt aus.
   ''' </summary>
   ''' <param name="sender">Objekt, welches gesendet hat.</param>
   ''' <param name="e">Ereignis Argumente.</param>
   ''' <remarks></remarks>
   Private Sub Clock_Tick(ByVal sender As Object, ByVal e As System.EventArgs) Handles _objClock.Tick
      ShowClock()                                                                            ' Datum und Zeit ausgeben. 
   End Sub

   '--------------------------------------------------------------------------------------------------------------------------------------------------
   ' Methode ShowClock
   '-----------------------------------------------------------------------------------------+--------------------------------------------------------
   ''' <summary>
   ''' Datum und Zeit ausgeben
   ''' </summary>
   ''' <remarks></remarks>
   Private Sub ShowClock()
      Me.tslClock.Text = System.DateTime.Now.ToString                                        ' Datum und Zeit ausgeben
   End Sub

#End Region

End Class