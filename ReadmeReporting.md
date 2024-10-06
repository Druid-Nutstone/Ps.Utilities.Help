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
&nbsp;
## Cmdlet Summary (Excel - OpenXml)
   | Cmdlet | Description | 
   | --- | --- | 
   | [New-Excel](#new-excel) | Creates the a new Excel application and returns a pointer to it.
   | [Add-ExcelStyle](#add-excelstyle) | Adds a style that that be used to format a row/cell
   | [Add-ExcelRow](#add-excelrow) | Add a row to the specified or current worksheet 
   | [Add-ExcelColumn](#add-excelcolumn) | Add a column to the specified or current worksheet row
   | [Add-WorkSheet](#add-worksheet) | Add's a new worksheet the the excel application   
   | [Set-WorkSheet](#set-worksheet) | Sets the properties of the specified worksheet     

# New-Excel
Creates a new excel (openxml) application and returns a pointer to it.

```
  $excelApp = New-Excel -WorkSheet "worksheet1"
```

### Parameters

__-WorkSheet__ (optional)

The name of the primary wotksheet to create

### Returns

a Type of ExcelAppliation

# Add-ExcelStyle
creates and adds a new formatting style to the specified ExcelApplication for use when formating specific rows and cells

```
  $excelApp | Add-ExcelStyle -Name "ErrorCell" `
                             -FontName "Calibri" `
                             -FontSize 12 `
                             -ForegroundColour Black `
                             -BackgroundColour Red `
                             -Format General `
                             -HorizontalAlignment Left `
                             -VerticalAlignment Bottom
```

### Parameters

__-Application__ (required - from pipeline)

The instance of the excel application

__-Name__ (required)

the [string] name of the style

__-FontName__ (optional)

The name of a valid system font.

__-FontSize__ (optional)

(double) size of the font.

__ForegroundColour__ (optional) 

(System,Drawing.Color) colour of the foreground font

__-BackgroundColour__ (optional) 

(System,Drawing.Color) colour of the background font

__-Format__ (optional)

Cell Format enum value of 

- General
- Decimal
- Date
- Currency 
- Percent

__-HorizontalAlignment__ (optional)

Cell horizontal alignment Enum value of 

- Left
- Center 
- Right  

__-VerticalAlignment__ (optional)

- Bottom
- Center
- Top


### Returns

a Type of ExcelAppliation

# Add-ExcelRow
Adds a new row excel (openxml) row to the current (or specified) worksheet it.

```
  $excelApp = New-Excel -WorkSheet "worksheet1"
```

### Parameters

__-Application__ (required - from pipeline)

The instance of the excel application

__-WorkSheet__ (optional)

The name of the target worksheet

### Returns

a Type of ExcelAppliation

# Add-ExcelColumn
Adds a new excel column (openxml) to the current (or specified row) at the current (or specified index)

```
  $excelApp | New-ExcelColumn -RowIndex 6 -ColumnIndex 4 -WorkSheet "worksheet1" -Value "row 6 cell 4" -Style "Hightlight"
```

### Parameters

__-Application__ (required - from pipeline)

The instance of the excel application

__-WorkSheet__ (optional)

The name of the target worksheet

__-RowIndex__ (optional - [int])

Specifies the row index to insert the column. if not specified the column is added to the 'current row'

__-ColumnIndex__ (optional - [int])

Specified the target column index to add the column. If not specified, the column is added to the cuurent row's next avalable column.

__-Value__ (required - [string])

The value to add to the cell (column)

__-Style__ (optional)

The [string] name of the style to add to the column (**see Add-ExcelStyle**)

### Returns

a Type of ExcelAppliation

# Add-WorkSheet
Adds a new excel worksheet the to ExcelApplication

```
  $excelApp | New-Worksheet -WorkSheet "Worksheet2"
```

### Parameters

__-Application__ (required - from pipeline)

The instance of the excel application

__-WorkSheet__ (required)

The name of the worksheet

### Returns

a Type of ExcelAppliation

# Set-WorkSheet
Set's specific properties to the specified worksheet (or worksheets)

```
  $excelApp | Set-Worheet -WorkSheet "Worksheet1" -AutoFit -FreezeTopRow
```

### Parameters

__-Application__ (required - from pipeline)

The instance of the excel application

__-WorkSheet__ (optional - [string])

The name of a single worksheet.

__-WorkSheets__ (optional - [string[]])

A string array of worksheet names to apply the properties to,.

__-AutoFit__ (optional SwitchParameter)

If specified attempts to 'fit' the columns to the maximum width of a column/row

__-FreezeTopRow__ (ootional SwitchParameter)

If specified freezes the top row so that it soes not move when the user scrools the worksheet

### Returns

a Type of ExcelAppliation