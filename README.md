####################################################
####### Getting all the views into CSV File ########
####################################################


Connect-PnPOnline -Url "Sourse Site" -UseWebLogin

$allViews = Get-PnPView -List "Sourse List"
 
foreach($View in $allViews)
{ 
   $out =""
    $fileds = ""
   foreach($fd in $View.ViewFields){  
    $out += "$fd,"
    } 
    $fileds = $out.Substring(0,$out.Length-1)

   $SCRows = New-Object -TypeName PSObject
   $SCRows | Add-Member -MemberType NoteProperty -Name "Title" -Value $View.Title -PassThru -Force
   $SCRows | Add-Member -MemberType NoteProperty -Name "Fields" -Value $fileds -PassThru -Force
   $SCRows | Add-Member -MemberType NoteProperty -Name "Query" -Value $View.ViewQuery -PassThru -Force
   $SCRows | Add-Member -MemberType NoteProperty -Name "RowLimit" -Value $View.RowLimit -PassThru -Force
   $SCRows | Add-Member -MemberType NoteProperty -Name "ViewType" -Value $View.ViewType -PassThru -Force
   $SCRows | Add-Member -MemberType NoteProperty -Name "SetAsDefault" -Value $View.DefaultView -PassThru -Force
   $Results += $SCRows 
   }
$Results| Export-Csv "C:\AllViews.csv" -NoTypeInformation

####################################################
######## Creatign Views using the CSV File #########
####################################################

Connect-PnPOnline -Url "Destination URL" -UseWebLogin

 
Import-Csv "C:\AllViews.csv" | ForEach-Object {   
    $fields = @()
    $fieldsArray = $_.Fields.Split(",") | ForEach-Object {
    $fields += $_
    }
    write-host $fields
    $Title = $_.Title  
    $Fields = $fields
    $Query = $_.Query  
    $RowLimit = $_.RowLimit  
    $ViewType = $_.ViewType  
    $SetAsDefault = $_.SetAsDefault 
    Add-PnPView -List "Destination List" -Title $Title -Fields $Fields -Query $Query -RowLimit $RowLimit -ViewType $ViewType
}
