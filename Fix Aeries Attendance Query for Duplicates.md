# Student Attendance Data Cleaning and ISP Data

Students that have been enrolled in ISP of any form have been showing up in duplicates in the Aeries query for attendance. Using this code, students that have duplicates will be cleaned by:
1) Separating them from the non duplicates.
2) Determining if they have completed, are in progress, or are incomplete in ISP.
3) Data will then be cleaned up depending on what category they are under.

Once the data has been cleaned for ISP students they will be rejoined with the rest of the students to produce a master attendance file that no longer has duplicates along with other sheets that give calculations on the number of students for each category at all of the school sites along with a list of students in progress and incomplete.

The Aeries query to obtain the data used for this file is (adjust the year in the query for the current school year):

LIST AHS STU STU.SC STU.NM STU.ID STU.GR AHS.YR AHS.EN AHS.PR AHS.AB AHS.AE AHS.AU AHS.TD AHS.TRU AHS.SU AHS.ISS AHS.ISC AHS.ISI IF AHS.YR = 2022-2023

__Make sure to scroll to the bottom to look over the checks on possible special situations that you might have to look into about the data.__


```python
import numpy as np
import pandas as pd
```


```python
# Copy the file pathway from the Aeries query into the parenthesis
attendance = pd.read_excel(r"C:\Users\derek.castleman\Desktop\AttendanceFile.xlsx")
attendance
```


```python
a = input('What is the school year interest (2021-2022, 2022-2023, etc.):          ')
```


```python
#Making sure that we are only looking at current school year
att = attendance[attendance['Year']== a]
att
```


```python
# Check to see if any students have more than two rows of data
more_than_two = att.groupby(['Student ID']).size()
more_than_two = more_than_two.to_frame()
more_than_two = more_than_two.rename(columns = {0:'Number Rows'}, inplace = False)
more_than_two = more_than_two[more_than_two['Number Rows'] > 2]
more_than_two
```


```python
#Separating duplicated students from the rest and keeping both rows for them
duplicated_students = att[att.duplicated(subset=['Student ID'], keep = False)]
duplicated_students
```

## Getting Students with Duplicated Rows and ISP Attendance

Duplicated students have two rows for them. They have the ones in which they are doing ISP and then their regular attendance at the school In this section, the students are sorted by their Student ID and then enrollment. The lower value represents the days in which they are in ISP so this will be the one that is selected. 

This is done by enrollment since some duplicated students are in home hospital and have no ISP days, so this method will capture these students as well.


```python
# Sorting students by ID and enrollment
students_sorted = duplicated_students.sort_values(['Student ID','Enrolled'])
students_sorted
```


```python
# The lowest value row for each student is selected
lowest_number = students_sorted.groupby(['Student ID']).head(1)
lowest_number
```


```python
# Present column is updated by adding in added days of independent study and incomplete which causes a negative number
lowest_number['Present'] = lowest_number['Present'] + lowest_number['Days of Complete Independent Study'] + lowest_number['Days of Incomplete Independent Study']
lowest_number
```

## Combining Students and Fixing Duplicates

Combining the data that has been fixed in the previous sections. Then concat it to the other duplicates that were filtered out at the beginning. Then merge the rows for each of the students to finally have the data fixed.


```python
students_sorted
```


```python
# Getting the other duplicate rows that were filtered out at first
highest_number = students_sorted.groupby(['Student ID']).tail(1)
highest_number
```


```python
# Updates present column with days of completed independent study and incomplete since these can be a negative
highest_number['Present'] = highest_number['Present'] + highest_number['Days of Complete Independent Study'] + highest_number['Days of Incomplete Independent Study']
highest_number
```


```python
# Combining data to recreate the duplicated rows for each student
combined_duplicated = pd.concat([highest_number, lowest_number]).sort_values(['Student ID','Enrolled'], ascending=False)
combined_duplicated
```


```python
# Merging the two rows for each student to create one single entry with the corrected data
Schoolfixed_absent = combined_duplicated.groupby(['Student Name','Grade','Student ID', 'School', 'Year']).sum().reset_index()
```


```python
Schoolfixed_absent
```


```python
# Checking to see if any unduplicated rows still remain
duplicated_students = Schoolfixed_absent[Schoolfixed_absent.duplicated(subset=['Student ID'], keep = False)]
duplicated_students
```

## Attendance with Single Line for Students

The file generated here has one row for each student with the ISP corrected as days present for the student for cases in which we want to see the full attendance for a student.


```python
# Generates a csv file from the final dataframe
import base64
from IPython.display import HTML

def create_download_link( df, title = "Attendance Fixed", filename = "Attendance Fixed"):
    csv = df.to_csv()
    b64 = base64.b64encode(csv.encode())
    payload = b64.decode()
    html = '<a download="{filename}" href="data:text/csv;base64,{payload}" target="_blank">{title}</a>'
    html = html.format(payload=payload,title=title,filename=filename)
    return HTML(html)

create_download_link(Schoolfixed_absent)
```

## Attendance with ISP Separated

The file generated here has the ISP rows still separated but are now corrected so that the completed study shows as being days present and the incomplete days no longer show as a negative number on attendance.


```python
# Generates a csv file from the final dataframe
import base64
from IPython.display import HTML

def create_download_link( df, title = "Attendance ISP Fixed", filename = "Attendance ISP Fixed"):
    csv = df.to_csv()
    b64 = base64.b64encode(csv.encode())
    payload = b64.decode()
    html = '<a download="{filename}" href="data:text/csv;base64,{payload}" target="_blank">{title}</a>'
    html = html.format(payload=payload,title=title,filename=filename)
    return HTML(html)

create_download_link(combined_duplicated)
```


```python

```
