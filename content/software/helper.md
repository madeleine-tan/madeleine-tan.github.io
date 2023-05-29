+++
title = "Helper code"
weight = 1
hideAuthor = true
hideDate = true
hideLastMod = true
+++

```python
# Renames sac files within subdirectories by their julian date
# Makes sure file follows YYYY.DDD.hh.mm.ss.sac

rootdir = '/Users/madeleinetan/Research/TA_ARRAYS/arrays_downloaded_112422/'
for i in os.listdir(rootdir):
    enter_dir = os.path.join(rootdir, i)
    if os.path.isdir(enter_dir):
        for path in os.listdir(enter_dir):

            try:
                st = read(os.path.join(enter_dir, path))
            except TypeError:
                print("File format error for file: ", path)
                break

            network = str(st[0].stats.network)
            station = str(st[0].stats.station)
            directory_name = network + "." + station

            if not os.path.exists(os.path.join(rootdir, directory_name)):
                os.makedirs(directory_name)

            year = str(st[0].stats.sac.nzyear)
            jday = str(st[0].stats.sac.nzjday)
            if len(jday) != 3:
                missing = 3 - len(jday)
                if missing == 2:
                    jday_final = str("00") + jday
                elif missing == 1:
                    jday_final = str("0") + jday
                elif missing == 3:
                    jday_final = str("000")
            else:
                jday_final = jday
            hour = str(st[0].stats.sac.nzhour)
            if len(hour) != 2:
                missing = 2 - len(hour)
                if missing == 1:
                    hour_final = str("0") + hour
                elif missing == 2:
                    hour_final = str("00")
            else:
                hour_final = hour
            min = str(st[0].stats.sac.nzmin)
            if len(min) != 2:
                missing = 2 - len(min)
                if missing == 1:
                    min_final = str("0") + min
                elif missing == 2:
                    min_final = str("00")
            else:
                min_final = min
            sec = str(st[0].stats.sac.nzsec)
            if len(sec) != 2:
                missing = 2 - len(sec)
                if missing == 1:
                    sec_final = str("0") + sec
                elif missing == 2:
                    sec_final = str("00")
            else:
                sec_final = sec
            comp = str(st[0].stats.sac.kcmpnm)
            newname = year + "." + jday_final + "." + hour_final + "." + min_final + "." + sec_final + "." + comp + ".sac"
            os.rename(os.path.join(enter_dir, path), os.path.join(enter_dir, newname))
            shutil.move(os.path.join(enter_dir, newname), os.path.join(rootdir, directory_name))
```
