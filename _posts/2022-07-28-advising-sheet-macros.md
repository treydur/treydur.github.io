---
layout: "post"
subtitle: ""
background: '/img/posts/advising-sheet-macros/excel_vba_background_post.png'
---

# Background
My employer wanted a macro to automatically populate courses into a separate table based on whether the course was recommended. The motivation behind this task was to emulate the school-sponsored software degreeworks in excel. The benefit of moving it to excel is that it is more shareable and updates quicker for real time collaboration. This simple macro would make the process much easier for the advisors in the department who can save time by avoid the repetitive task of copying and pasting recommended courses for thousands of students.

![Course Table]({{site.url}}/img/posts/advising-sheet-macros/advising_table_post.png){:.img-fluid}

# Problem
This program needed to run on 27 different versions of the same excel sheet and be robust enough to sustain formatting changes. Xlookup might have worked if the lookup values were kept in contiguous cells. In this case the cells would be split into two separate sections, where when one overflows the other is filled.
![Recommended Table]({{site.url}}/img/posts/advising-sheet-macros/recommended_table_post.png){:.img-fluid}
<span class="caption text-muted">Cells where the recommended course are populated</span>

# Solution
Of course, programming the solution was simple: search, save, and populate. However, there was much more to be done to ensure that (A) it was robust enough, (B) easy enough to input into 27 different sheets and (C) fast.
(A) and (B) I was able to takle two birds with one stone by using named ranges. It was given that the cells among the different spreadsheet versions wouldn't match up, so it was important that I use named ranges in the program to avoid having to modify the code for each sheet. The only manual part that had to be done was naming the ranges I needed on each sheet. Not only does this make the code easier to follow, it makes it flexible for formatting changes.
### Optimization
(C) The first versions of the program were frankly ugly. I came from java and was trying to apply object oriented principles to a scripting language. I defined my own classes and types, kept the animations on, and populated each cell individually as it was found. All of this wasn't necessary. User defined types are inherently slower in excel and certain features don't need to be on during runtime. Lastly, the most shocking truth I learned was that it is faster to ["read/write large blocks of cells in a single operation".][1]
<pre>
'Read all the values at once into arrays
DataRange = Range("DataRange").Value2
RecoRange1 = Range("RecoRange1").Value2
RecoRange2 = Range("RecoRange2").Value2
...
    filter values
                     ...
'Write all the values from the arrays at once
Range("RecoRange1").Value2 = RecoRange1
Range("RecoRange2").Value2 = RecoRange2
</pre>

[1]: https://www.microsoft.com/en-us/microsoft-365/blog/2009/03/12/excel-vba-performance-coding-best-practices/ "Excel vba best practices"

### Automation
Whenever the sheet undergoes a change in the specified names range, the macro is automatically run.
<pre>
Private Sub Worksheet_Change(ByVal Target As Range)
On Error GoTo ErrHandler
    
    If Not Intersect(Target, Range("DataRange")) Is Nothing Then
        Application.EnableEvents = False
        Application.OnTime Now, "fillTable"
        Debug.Print "DataRange has been changed"
    End If

ErrHandler:
    Application.EnableEvents = True
End Sub
</pre>

# Remarks
With more time I could have written a script to automatically import this same code into each of the 27 sheets, saving each more time and easing the pressure off of having perfect code. I hadn't used acros prior to this tasks or ever seen visual basic, but can confidently say that now I am more comfortable with macros, visual basic syntax, and the libraries used in VBA. 
