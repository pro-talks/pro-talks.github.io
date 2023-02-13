---
layout: post
title: "Python - How generate xlsx fast?"
date: 2023-02-13 12:00:00 +0100
categories: python xlsx
permalink: python/generate-xlsx
---

# Python - How generate xlsx fast?
For example, you need to generate xlsx file with styled header and footer, 10 columns, like here

![]({{ site.baseurl }}/img/python/generate_xlsx.png)

## Optimization 
If the file is small, this won't be a problem.
But if there are more than 100k lines in the file.
Then you need to think about optimization!

## Result for file 10 columns x 200_000 rows

1. openpyxl, time is **242.06575560569763**
2. openpyxl_optimization_mode, time is **256.0805859565735**
3. openpyxl_optimization_mode_plus_defaul_border, time is **80.0022509098053**
4. pyexcelerate, time is **63.69864106178284**
5. pyexcelerate_optimization_mode, time is **58.15235757827759**
6. pyexcelerate_optimization_mode_without_default_borders_if_file_is_huge, time is **29.15982484817505**


## Prepare data
```python
def prepare_data(n_rows):
    data = {}

    rows = []
    for row_index in range(n_rows):
        rows.append(['01.01.2023', 1234, 1.234, 'Description with date 2023.01.19 17:28',
       'Address', '123-4567-89', '12', '5.00', '7.00', '20.00%'])

    data['headers'] = ['Date', 'Number1', 'Number2', 'Description', 'Location', 'Code', 
                       'value1 %', 'value2 %', 'value3 %', 'value4 %']
    data['footer'] = data['headers']
    data['rows'] = rows

    return data
```

## Library "openpyxl"
```python
def generate_xlsx_openpyxl(data: dict) -> io.BytesIO:

    BORDER_MEDIUM = Border(
        left=Side(color='00000000', style='medium'),
        top=Side(color='00000000', style='medium'),
        right=Side(color='00000000', style='medium'),
        bottom=Side(color='00000000', style='medium'),
    )
    BORDER_THIN = Border(
        left=Side(color='00000000', style='thin'),
        top=Side(color='00000000', style='thin'),
        right=Side(color='00000000', style='thin'),
        bottom=Side(color='00000000', style='thin'),
    )

    wb = openpyxl.Workbook()
    ws = wb.active
    
    # Set column's width
    for letter in ['A', 'B', 'C', 'D', 'E', 'F', 'G']:
        ws.column_dimensions[letter].width = 14

    # Set style for headers
    for col, value in enumerate(data['headers'], start=1):
        cell = ws.cell(row=1, column=col, value=value)
        cell.style = 'Headline 4'
        cell.border = BORDER_MEDIUM

    for row_index, row in enumerate(data['rows'], start=1):
        for col, value in enumerate(row, start=1):
            cell = ws.cell(row=1 + row_index, column=col, value=value)
            cell.border = BORDER_THIN

    # Set style for footer
    row_index = 1 + len(data['rows'])
    for col, value in enumerate(data['footer'], start=1):
        cell = ws.cell(row=row_index, column=col, value=value)
        cell.style = 'Headline 4'
        cell.border = BORDER_MEDIUM

    output = io.BytesIO()
    wb.save(output)

    return output
```

## Library "openpyxl" in optimization mode

1. Create `Workbook` with special flag `write_only=True`
2. Use special class `WriteOnlyCell`

```python
def generate_xlsx_openpyxl_optimization_mode(data: dict) -> io.BytesIO:

    BORDER_MEDIUM = Border(
        left=Side(color='00000000', style='medium'),
        top=Side(color='00000000', style='medium'),
        right=Side(color='00000000', style='medium'),
        bottom=Side(color='00000000', style='medium'),
    )
    BORDER_THIN = Border(
        left=Side(color='00000000', style='thin'),
        top=Side(color='00000000', style='thin'),
        right=Side(color='00000000', style='thin'),
        bottom=Side(color='00000000', style='thin'),
    )

    wb = openpyxl.Workbook(write_only=True)
    ws = wb.create_sheet()
    for letter in ['A', 'B', 'C', 'D', 'E', 'F', 'G']:
        ws.column_dimensions[letter].width = 14

    headers = []
    for col, value in enumerate(data['headers'], start=1):
        cell = WriteOnlyCell(ws, value=value)
        cell.style = 'Headline 4'
        cell.border = BORDER_MEDIUM
        headers.append(cell)
    ws.append(headers)

    for row_index, row in enumerate(data['rows'], start=1):
        write_row = []
        for col, value in enumerate(row, start=1):
            cell = WriteOnlyCell(ws, value=value)
            cell.border = BORDER_THIN
            write_row.append(cell)
        ws.append(write_row)

    footer = []
    for col, value in enumerate(data['footer'], start=1):
        cell = WriteOnlyCell(ws, value=value)
        cell.style = 'Headline 4'
        cell.border = BORDER_MEDIUM
        footer.append(cell)
    ws.append(footer)

    output = io.BytesIO()
    wb.save(output)

    return output
```

## Library "openpyxl" in optimization mode + default borders
```python
def generate_xlsx_openpyxl_optimization_mode_plus_defaul_border(data: dict) -> io.BytesIO:

    BORDER = Border()

    BORDER_MEDIUM = Border(
        left=Side(color='00000000', style='medium'),
        top=Side(color='00000000', style='medium'),
        right=Side(color='00000000', style='medium'),
        bottom=Side(color='00000000', style='medium'),
    )
    BORDER_THIN = Border(
        left=Side(color='00000000', style='thin'),
        top=Side(color='00000000', style='thin'),
        right=Side(color='00000000', style='thin'),
        bottom=Side(color='00000000', style='thin'),
    )

    wb = openpyxl.Workbook(write_only=True)
    ws = wb.create_sheet()
    for letter in ['A', 'B', 'C', 'D', 'E', 'F', 'G']:
        ws.column_dimensions[letter].width = 14

    # add default border
    for letter in ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J']:
        ws.column_dimensions[letter].border = BORDER_THIN

    headers = []
    for col, value in enumerate(data['headers'], start=1):
        cell = WriteOnlyCell(ws, value=value)
        cell.style = 'Headline 4'
        cell.border = BORDER_MEDIUM
        headers.append(cell)
    ws.append(headers)

    for row_index, row in enumerate(data['rows'], start=1):
        ws.append(row)

    footer = []
    for col, value in enumerate(data['footer'], start=1):
        cell = WriteOnlyCell(ws, value=value)
        cell.style = 'Headline 4'
        cell.border = BORDER_MEDIUM
        footer.append(cell)
    ws.append(footer)

    output = io.BytesIO()
    wb.save(output)

    return output
```

## Library "pyexcelerate"
```python
def generate_xlsx_pyexcelerate(data: dict) -> io.BytesIO:

    BORDER_MEDIUM = Borders.Borders(
        top = PBorder.Border(color=Color.BLACK, style='medium'),
        left = PBorder.Border(color=Color.BLACK, style='medium'),
        right = PBorder.Border(color=Color.BLACK, style='medium'),
        bottom = PBorder.Border(color=Color.BLACK, style='medium')
    )

    BORDER_THIN = Borders.Borders(
        top = PBorder.Border(color=Color.BLACK, style='thin'),
        left = PBorder.Border(color=Color.BLACK, style='thin'),
        right = PBorder.Border(color=Color.BLACK, style='thin'),
        bottom = PBorder.Border(color=Color.BLACK, style='thin')
    )

    HEADERS = Style(
        borders=BORDER_MEDIUM,
        font=Font(color=Color(31, 73, 125, 255), bold=True)
    )

    CELL = Style(
        borders=BORDER_THIN
    )

    wb = PWorkbook()
    ws = wb.new_sheet('Sheet')
    for col in range(1, 8):
        ws.set_col_style(col, Style(size=14))

    for col, value in enumerate(data['headers'], start=1):
        ws.set_cell_value(1, col, value)
        ws.set_cell_style(1, col, HEADERS)

    for row_index, row in enumerate(data['rows'], start=1):
        for col, value in enumerate(row, start=1):
            ws.set_cell_value(1 + row_index, col, value)
            ws.set_cell_style(1 + row_index, col, CELL)

    row_index = len(data['rows'])
    for col, value in enumerate(data['footer'], start=1):
        ws.set_cell_value(1 + row_index, col, value)
        ws.set_cell_style(1 + row_index, col, HEADERS)

    output = io.BytesIO()
    wb.save(output)

    return output
```

## Library "pyexcelerate" and the fastest way to write
```python
def generate_xlsx_pyexcelerate_optimization_mode(data: dict) -> io.BytesIO:

    BORDER_MEDIUM = Borders.Borders(
        top=PBorder.Border(color=Color.BLACK, style='medium'),
        left=PBorder.Border(color=Color.BLACK, style='medium'),
        right=PBorder.Border(color=Color.BLACK, style='medium'),
        bottom=PBorder.Border(color=Color.BLACK, style='medium')
    )

    BORDER_THIN = Borders.Borders(
        top=PBorder.Border(color=Color.BLACK, style='thin'),
        left=PBorder.Border(color=Color.BLACK, style='thin'),
        right=PBorder.Border(color=Color.BLACK, style='thin'),
        bottom=PBorder.Border(color=Color.BLACK, style='thin')
    )

    HEADERS = Style(
        borders=BORDER_MEDIUM,
        font=Font(color=Color(31, 73, 125, 255), bold=True)
    )

    CELL = Style(
        borders=BORDER_THIN
    )

    arr = [data['headers']] + data['rows'] + [data['footer']]

    wb = PWorkbook()
    ws = wb.new_sheet('Sheet', arr)

    # Set styles
    for col in range(1, 8):
        ws.set_col_style(col, Style(size=14))

    for col, value in enumerate(data['headers'], start=1):
        ws.set_cell_style(1, col, HEADERS)

    for row_index, row in enumerate(data['rows'], start=1):
        for col, value in enumerate(row, start=1):
            ws.set_cell_style(1 + row_index, col, CELL)

    row_index = len(data['rows'])
    for col, value in enumerate(data['footer'], start=1):
        ws.set_cell_style(1 + row_index + 1, col, HEADERS)

    output = io.BytesIO()
    wb.save(output)

    return output
```


## Library "pyexcelerate" and the fastest way to write + ignore border if file is huge
```python
def pyexcelerate_optimization_mode_ignore_borders_if_file_is_huge(data: dict) -> io.BytesIO:

    BORDER_MEDIUM = Borders.Borders(
        top=PBorder.Border(color=Color.BLACK, style='medium'),
        left=PBorder.Border(color=Color.BLACK, style='medium'),
        right=PBorder.Border(color=Color.BLACK, style='medium'),
        bottom=PBorder.Border(color=Color.BLACK, style='medium')
    )

    BORDER_THIN = Borders.Borders(
        top=PBorder.Border(color=Color.BLACK, style='thin'),
        left=PBorder.Border(color=Color.BLACK, style='thin'),
        right=PBorder.Border(color=Color.BLACK, style='thin'),
        bottom=PBorder.Border(color=Color.BLACK, style='thin')
    )

    HEADERS = Style(
        borders=BORDER_MEDIUM,
        font=Font(color=Color(31, 73, 125, 255), bold=True)
    )

    CELL = Style(
        borders=BORDER_THIN
    )

    arr = [data['headers']] + data['rows'] + [data['footer']]

    wb = PWorkbook()
    ws = wb.new_sheet('Sheet', arr)

    # Set styles
    for col in range(1, 8):
        ws.set_col_style(col, Style(size=14))

    for col, value in enumerate(data['headers'], start=1):
        ws.set_cell_style(1, col, HEADERS)

    # setting styles costs when file is huge
    big_file = len(data['rows']) > 1000

    if not big_file:
        for row_index, row in enumerate(data['rows'], start=1):
            for col, value in enumerate(row, start=1):
                ws.set_cell_style(1 + row_index, col, CELL)

    row_index = len(data['rows'])
    for col, value in enumerate(data['footer'], start=1):
        ws.set_cell_style(1 + row_index + 1, col, HEADERS)

    output = io.BytesIO()
    wb.save(output)

    return output
```

## Conclusions

1. If you need to generate huge xlsx files then library "pyexcelerate" is the good choose.
2. Setting styles costs when file is huge, try to set default styles for whole file and not for every cell.


<h2 id="Download file with the examples"><a href="/download/100-python/generate_xlsx_fast.py">Download file with the examples</a>
