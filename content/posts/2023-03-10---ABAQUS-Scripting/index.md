---
title: "Unlocking Efficiency: ABAQUS Scripting"
date: "2023-03-10"
template: "post"
draft: false
slug: "/posts/abaqus-scripting"
category: "Mechanical Engineer"
tags:
  - "CAE"
  - "ABAQUS"
description: "Discover the power of ABAQUS scripting and revolutionize your engineering simulations with automation, customization, and enhanced efficiency. "
socialImage: "./media/notebook.jpg"
---

In the world of engineering simulations, ABAQUS CAE has established itself as a powerful tool for finite element analysis (FEA) and multiphysics simulations. Its user-friendly graphical interface allows engineers and researchers to build complex models, define material properties, and analyze structural behavior with relative ease. However, when it comes to repetitive or intricate tasks, manually navigating the graphical interface can be time-consuming and cumbersome.

This is where ABAQUS scripting comes into play, offering a way to automate and customize simulations, streamline workflows, and ultimately enhance productivity. By harnessing the power of scripting, engineers can tap into the full potential of ABAQUS, unlocking new possibilities and gaining a competitive edge in the field of numerical analysis.

Scripting in ABAQUS involves writing lines of code in a supported scripting language, such as Python, to interact with the software's underlying functionality. This allows users to automate repetitive tasks, control simulation parameters, create customized post-processing routines, and even develop advanced simulation workflows. With scripting, engineers can go beyond the limitations of the graphical user interface and tailor ABAQUS to their specific needs.

## Data Extraction and Filter
During my work on my final thesis, I realized that for a whole week, I was repeatedly performing the same task. It was a painful and stressful job that required a great deal of focus to avoid making mistakes and having to start over from scratch. This story definitely calls for a follow-up post, but for now, let's focus solely on the beginning of my journey through ABAQUS scripting.

After spending a week doing manual work, I thought, why not invest one or two days in trying to automate the entire process? At the start, I was quite naive. The plan I had in mind seemed very robust, but as we all know, things like these rarely go as planned. The first step of my foolproof plan involved extracting the stress tensor components for each node of my model during the different time instances of the simulation. It sounded pretty simple, so why would you need to script something like this? After conducting extensive research on dubious forums and consulting old Dassault documentation, I discovered that there simply isn't an option to extract all the values at once. Instead, you have to repeatedly click the button manually.

Later on, I would find out that there are many seemingly simple tasks that cannot be accomplished through the ABAQUS GUI interface. It's time to unleash the full power of the software!

The command that allows us to write a field output report to a file is called `writeFieldReport()`. Generally, we are only interested in specific zones rather than the entire model. Therefore, before extracting the results, we must select the appropriate node set.

```python
from abaqus import *
from abaqusConstants import *
import displayGroupOdbToolset as dgo
from odbAccess import *
from driverUtils import *

#Open results file
odb = session.openOdb(name=Job-1.odb)
session.viewports['Viewport: 1'].setValues(displayedObject=odb)

#Filter Nodes
leaf = dgo.LeafFromNodeSets(nodeSets=(node_set, ))
session.viewports['Viewport: 1'].odbDisplay.displayGroup.replace(leaf=leaf)
```

Now, we can iterate the function for all the frames of the simulation:
```python
# Extract Time Points, all_frames contains all the time points including the artificials one
step = odb.steps[name_step]
t1 = step.totalTime # time till the 'Step-1'

for frame in step.frames:
    
        tinst = frame.frameValue   # time for the frame in the current step
        tcurrent = t1 + tinst
        all_frames = np.append(all_frames, round(tcurrent,3))


for i in range(len(frame_numbers)):
  session.writeFieldReport(fileName="data_" + Job1 + ".csv", append=ON,
                          sortItem='Node Label', odb=odb, step=0, frame=frame_numbers[i],
                          outputPosition=NODAL,
                          variable=(('S', INTEGRATION_POINT, ((COMPONENT, 'S11'),
                                                                (COMPONENT, 'S22'),
                                                                (COMPONENT, 'S33'),
                                                                (COMPONENT, 'S12'),
                                                                (COMPONENT, 'S13'),
                                                                (COMPONENT, 'S23'),
                                                                (INVARIANT, 'Mises'),
                                                                )),))

```

The result is a massive and disorganized CSV file that not only contains the information we need but also a lot of irrelevant data. It is imperative that we filter it.
```python
csv_file_path = "data_" + Job1 + ".csv"


# define the column indices to be deleted
columns_to_delete = [0,1,3,5,6,7,8,9,10]

# define the string to match at the beginning of rows
string_to_match = 'ODB Name'

# open the CSV file and read its contents
with open(csv_file_path, 'r') as csv_file, open("filtered_" + Job1 + ".csv", 'w') as output_file:
    reader = csv.reader(csv_file)
    writer = csv.writer(output_file)

    # iterate over each row in the input file
    for row in reader:
        # check if the row starts with the string to match
        if not row[0].startswith(string_to_match):
            # remove the specified columns from the row
            filtered_row = [value for i, value in enumerate(row) if i not in columns_to_delete]

            # convert the numerical columns in scientific notation to float
            for i in range(len(filtered_row)):
                try:
                    filtered_row[i] = float(filtered_row[i])
                except ValueError:
                    #filter the column of the frame
                    filtered_row[i] = float(filtered_row[i].split(' ')[-1])
                    pass

            # write the filtered row to the output file
            writer.writerow(filtered_row)

```

Finally, we obtain a simple text file that contains only the relevant information: node label, time, and stress tensor components. This process takes about 2 minutes and only requires the results file (.odb) and the name of the node set.