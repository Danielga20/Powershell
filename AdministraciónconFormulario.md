## Administración de servicio de directorio por Formulario


Lo que hace esta practica es que mostrara en pantalla un formulario con el Label "Operación" y "Nombre" y cuando en el TextBox "Operación" pongan OU(Unidad Organizativa) se creara un usuario en la unidad organizativa (en este caso mi unidad organizativa por defecto sera RecursosHumanos asi que debera estar creada previamente), cuando pongan Usuario creara el usuario en local y cuando pongan Grupo se metara el usuario de la unidad organizativa en el grupo Informatica que es el grupo de mi OU(RecursosHumanos).

Para que esta practica funcione deberemos tener XAMPP instalado y deberan activar "APACHE", aparte de esto el sript de a continuación debera ejecutarse en powershell estando en la ruta del xampp/htdocs

## Código Formulario

     using assembly System.Windows.Forms
     using namespace System.Windows.Forms
 
     class Operacion
     { 
      [String]$operacion
      [String]$nombre
  
    Operacion($operacion,$nombre)
    {
        $this.operacion = $operacion
        $this.nombre = $nombre
    }
    }
 
    $arrayoperaciones = New-Object System.Collections.ArrayList
 
    $form = [Form] @{
    Text = 'Mi formulario'


    }


    $Label=New-Object System.Windows.Forms.Label
    $Label.Text="Nombre"
    $Label.AutoSize=$True
    $Label.Location=New-Object System.Drawing.Size(38,55)
    $Label.BackColor = "BLUE"
    $Label.ForeColor = "WHITE"

    $TextBox = [TextBox] @{
    Location = New-Object System.Drawing.Size(40,80)
    }

    $Label2=New-Object System.Windows.Forms.Label
    $Label2.Text="Operación"
    $Label2.AutoSize=$True
    $Label2.Location=New-Object System.Drawing.Size(38,113)
    $Label2.BackColor = "GREEN"
    $Label2.ForeColor = "WHITE"

    $TextBox2 = [TextBox] @{
    Location = New-Object System.Drawing.Size(40,140)
    }
 
    $button = [Button] @{
    Text = 'Aceptar'
    Location = New-Object System.Drawing.Size(40,185)
    }

    $button2 = [Button] @{
    Text = 'Cerrar'
    Location = New-Object System.Drawing.Size(150,185)
    }

    $button.add_Click{
    Write-Host $TextBox.Text
    Write-Host $TextBox2.Text
    $arrayoperaciones.add([Operacion]::new($TextBox2.Text,$TextBox.Text))
    }

    $button2.Add_Click({$Form.Close()})

  
    $Form.Controls.Add($Label2)
    $Form.Controls.Add($Label)
    $form.Controls.Add($TextBox)
    $form.Controls.Add($button) 
    $form.Controls.Add($TextBox2)

    $form.Controls.Add($button2)
    $form.ShowDialog()

    $arrayoperaciones | ConvertTo-Json | Out-File operaciones.json -Append -Encoding default


## 2da parte - Realizar operaciónes


    $ficherojson = Invoke-RestMethod "http://localhost/operaciones.json" 

    $ficherojson

    foreach($operacion in $ficherojson)
    {
   
     if($operacion.operacion -eq "Usuario")
        {
         $pass = ConvertTo-SecureString -String “p@sswdor1” -Asplaintext -force
         new-localuser -name $operacion.nombre -password $pass 
        }
     elseif($operacion.operacion -eq "OU" )
       {
        New-ADUSer -Name $operacion.nombre -Sam $operacion.nombre -Path ("OU=RecursosHumanos,dc=andel,dc=local") `
        -AccountPassword (ConvertTo-SecureString "Secreto@123@Top" -AsPlainText -force) -Enable $true
        }
     elseif($operacion.operacion -eq "Grupo")
        {
         Add-ADGroupMember -Members $operacion.nombre -Identity Informatica 
        }
    
     }
