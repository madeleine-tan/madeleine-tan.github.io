+++
title = "Image rename GUI"
hideAuthor = true
hideDate = true
hideLastMod = true
+++

```python
import PySimpleGUI as sg
import os
from glob import glob
from datetime import datetime
from PIL import Image

def new_name(cat):
    try:
        date = str(get_date(str(values["{}".format(cat)])))[0:10]
        fn = (str(site + "_" + date + "_{}".format(cat) + ".jpg"))
        os.rename(str(values["{}".format(cat)]), os.path.join(os.path.dirname(str((values["{}".format(cat)]))), fn))
    except AttributeError:
        pass
    except:
        print("> Unable to rename {}.".format(cat), "Image needs to be in JPEG format.")


def get_date(fn):
    # credit to Stack Overflow
        # returns the image date from image (if available)\nfrom Orthallelous
    std_fmt = '%Y:%m:%d %H:%M:%S.%f'
    tags = [(36867, 37521),  # (DateTimeOriginal, SubsecTimeOriginal)
            (36868, 37522),  # (DateTimeDigitized, SubsecTimeDigitized)
            (306, 37520), ]  # (DateTime, SubsecTime)

    with Image.open(fn) as img:
        #exif = Image.open(fn)._getexif()
        exif = img._getexif()

        for t in tags:
            dat = exif.get(t[0])
            sub = exif.get(t[1], 0)

            # PIL.PILLOW_VERSION >= 3.0 returns a tuple
            dat = dat[0] if type(dat) == tuple else dat
            sub = sub[0] if type(sub) == tuple else sub
            if dat != None: break

        if dat == None: return None
        full = '{}.{}'.format(dat, sub)
        T = datetime.strptime(full, std_fmt)
    return T

sg.theme('LightGreen3')
file_types = ((".jpg"))

logwindow = sg.Multiline(size=(60, 10), font=('Courier', 10))
print = logwindow.print

layout = [  [sg.Text('Site name:')],
            [sg.InputText(key='site')],
            #[sg.Checkbox('Extract', size=(10,1), key='chk_extract'),
             #sg.Checkbox('Install', size=(10,1), key = 'chk_install'),
             #sg.Checkbox('Other', size=(10,1), key = 'chk_other')],
            [sg.Text('_' * 100, size=(65, 1))],
            [sg.Text('North')],
            [sg.In(key='N'), sg.FileBrowse(file_types=(file_types))],
            [sg.Text('East')],
            [sg.In(key='E'), sg.FileBrowse(file_types=(file_types))],
            [sg.Text('South')],
            [sg.In(key='S'), sg.FileBrowse(file_types=(file_types))],
            [sg.Text('West')],
            [sg.In(key='W'), sg.FileBrowse(file_types=(file_types))],
            [sg.Text('Magnetometer')],
            [sg.In(key='mag'), sg.FileBrowse(file_types=(file_types))],
            [sg.Text('NIMS')],
            [sg.In(key='NIMS'), sg.FileBrowse(file_types=(file_types))],
            [logwindow],
            [sg.Button('Ok'), sg.Button('Cancel')] ]

window = sg.Window('File rename', layout)

# Event Loop to process "events" and get the "values" of the inputs
while True:
    event, values = window.read()
    if event == sg.WIN_CLOSED or event == 'Cancel': # if user closes window or clicks cancel
        break
    if event == 'Ok':
        site = str(values["site"])
        new_name("N")
        new_name("E")
        new_name("S")
        new_name("W")
        new_name("mag")
        new_name("NIMS")
        print("> Code complete.")

window.close()
```
