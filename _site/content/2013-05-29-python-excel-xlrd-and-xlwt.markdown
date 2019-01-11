The other week we received a request for a small script from a department on campus. The department needed to evaluate data from a Microsoft Excel worksheet which they wrote a macro for. Their problem, though, was to aggregate all their data into one sheet. This aggregation process was not only the most tedious but was also their longest task taking nearly 2 weeks to complete. My coworkers and I were shocked that they've continued this process for so long without looking to automate it somehow, so we took on the challenge to ease the pain in their lives. 

I set out to write a Python script and immediately researched what Microsoft Excel libraries are available for Python. I quick found Python-Excel and the XLRD and XLWT libraries, which I have to say, have been nothing short but amazing to use. The libraries are intuitive and easy to use. There was not a function I could think of that this library didn't provide. The code can be found on [GitHub](https://github.com/barakyo/excel-extractor).

Just some quick examples of how I was using the libraries:

**Opening an excel file:** 
{{< highlight python >}}
try:
    # Open the book
    book = xlrd.open_workbook(file_name)
except IOError:
    logging.warning("File: " + file_name + " does not exist")
{{< / highlight >}}

**Opening a specific sheet:**
{{< highlight python >}}
# Try to open the specific sheet
try:
    summary_sheet = book.sheet_by_name('For Summary File')
except:
    logging.warning('Could not open sheet ' + sheet_name + ' in file ' + file_name)
{{< / highlight >}}

**Copying a row:**
{{< highlight python >}}
row = [summary_sheet.cell(1, col).value for col in range(summary_sheet.ncols)]
{{< / highlight >}}

**Creating a new workbook and sheet:**
{{< highlight python >}}
# Create a new workbook
new_workbook = xlwt.Workbook()
# Create a new sheet
new_sheet = new_workbook.add_sheet('Sheet1')
{{< / highlight >}}

**Writing data to the new workbook:**
{{< highlight python >}}
# Write the data to the sheet
new_sheet.write(row, col, label=current_row[col])
{{< / highlight >}}

**Saving a workbook:**
{{< highlight python >}}
new_workbook.save(output_file)
{{< / highlight >}}

Doesn't get much simpler than that! :)