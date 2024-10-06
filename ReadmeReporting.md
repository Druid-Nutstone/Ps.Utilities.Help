# PS.Reporting
P.Reporting is a set of C# cmdlets that wrap various reporting methods. It supports Excel (via open Xml) , HTML (in progress) and Csv (in progress). It will create reports in the various outputs from objects, datasets (ne tables) and from single row/cell functions.

## Using PS.Selenium 
```
   Install-Module PS.Reporting
   Import-Module PS.Reporting
```

# Examples 

## Excel (OpenXml) - Free form data input

```
$testPath = "C:\deleteme\test.xlsx"
$worksheetName = "David"
$workSheetName2 = "David 2"

Remove-Item -Path $testPath -ErrorAction Ignore

# create the app 
$excelApp = New-Excel -WorkSheet $worksheetName 

# add some global styles 
$excelApp | Add-ExcelStyle -Name "HighLight" -ForegroundColour Green -BackgroundColour AliceBlue `
          | Add-ExcelStyle -Name "HeaderRow" -ForegroundColour Black -BackgroundColour LightGray -Bold 
          
# add some rows and columns with data and styles to the 'current' worksheet           
$excelApp | Add-ExcelRow | Add-ExcelColumn -Value "Row 1 - column 1" -Style "HeaderRow" | Add-ExcelColumn -Value "Row 1 - Column 2" -Style "HeaderRow" `
          | Add-ExcelRow | Add-ExcelColumn -Value "Row 2 - column 2 - that has a very long column"  | Add-ExcelColumn -Value "Row 2 highlight" -Style "HighLight" 

# add new work sheet and make it the current worksheet 
$excelApp | Add-WorkSheet -WorkSheet $workSheetName2 

# add data to the new worksheet 
$excelApp | Add-ExcelRow | Add-ExcelColumn -Value "David 2 Row 1 - column 1" -Style "HeaderRow" | Add-ExcelColumn -Value "David 2 Row 1 - Column 2" -Style "HeaderRow" `
          | Add-ExcelRow | Add-ExcelColumn -Value "David 2 Row 2 - column 2 - that has a very long column"  | Add-ExcelColumn -Value "Row 2 highlight" -Style "HighLight" 


$excelApp | Set-Worksheet -WorkSheet $worksheetName -AutoFit -FreezeTopRow  
$excelApp | Set-Worksheet -WorkSheet $workSheetName2 -AutoFit -FreezeTopRow 



$excelApp | Save-To -Path $testPath 

Open-Excel -Path $testPath

```

## Excel - OpenXml from .net Dataset/DataTable  

```

# execute a couple of selects and return as a dataset
$workServer = "loadhost\servername"
$recordSet = Invoke-Sqlcmd -Query "select Id,Firstname,Surname,Gender,Dob from [data].[dbo].[table]; select * from [database].[dbo].[table]" -ServerInstance $workServer -TrustServerCertificate -OutputAs DataSet

$testPath = "C:\deleteme\test.xlsx"

# create string[] of worksheets 
$worksheets = "David 1","David 2"

Remove-Item -Path $testPath -ErrorAction Ignore

# create the app 
$excelApp = New-Excel 

# add some global styles 
$excelApp | Add-ExcelStyle -Name "HighLight" -ForegroundColour Green -BackgroundColour AliceBlue `
          | Add-ExcelStyle -Name "HeaderRow" -ForegroundColour Black -BackgroundColour LightGray -Bold `
          | Add-ExcelStyle -Name "Date" -Format Date -HorizontalAlignment Left


# write the datatables out to seperate worksheets 
# The -Onrow scriptblock will get executed every timne a row is created 
$excelApp | Write-SqlExcel -Data $recordSet -WorkSheets $worksheets -Onrow { 
   param($row, $rowIndex, $worksheetName)
   if ($rowIndex -eq 0) {
      # make top row a header style  
      $excelApp | Set-ExcelRowStyle -Row $row -Style "HeaderRow"
   }
   if ($rowIndex -gt 0 -and $worksheetName -eq "David 1") {
      # get the column index for the Id cell (column)
      $idIndex = $excelApp | Get-ExcelHeaderRowIndex -CellName "Id" -WorkSheet $worksheetName
      # get the value of the id column for the current row 
      $testValue = $excelApp | Get-ExcelCellValue -RowIndex $rowIndex -CellIndex $idIndex -WorkSheet $worksheetName
      if ($testValue -gt 28) {
         # hightlight the entire row 
         $excelApp | Set-ExcelRowStyle -Row $row -Style "HighLight"
      }
   }
} -OnCell { 
   param($cell, $rowIndex, $cellIndex, $cellName) 
   if ($rowIndex -gt 0 -and $cellName -eq "Dob") {
      # if the column is dob then format the cell to show the date (not the number)
      $excelApp | Set-ExcelCellStyle -Cell $cell -Style "Date"
   }
} 

# make autofit and freeze top row for all worksheets
$excelApp | Set-Worksheet -WorkSheets $worksheets -AutoFit -FreezeTopRow 

# save to disk (could be Base64 with -Base64 parameter)
$excelApp | Save-To -Path $testPath 

Open-Excel -Path $testPath

```
