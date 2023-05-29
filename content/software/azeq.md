+++
title = "Event map"
hideAuthor = true
hideDate = true
hideLastMod = true
+++

```python
from mpl_toolkits.basemap import Basemap
import matplotlib.pyplot as plt
import numpy as np
import os
import csv
from obspy import read, Stream
from datetime import date
import matplotlib as mpl

def bad_events(badfilepath):
    """
    Bad events to remove from map
    :param badfilepath: file path to bad event file
    :return: list of bad events
    """
    text_file = open(badfilepath, "r")
    ListToSort = text_file.read().splitlines()
    ListToSort = np.array(ListToSort)
    text_file.close()

    return ListToSort


def _fig0(title, lat_ref, lon_ref, lats, lons, M, path, grid=True, save=True):
    """
    Plot azimuthal equidistant map centered on reference coordinates
    :param lat_ref: reference latitude
    :param lon_ref: reference longitude
    :param lats: Numpy Array of latitudes
    :param lons: Numpy Array of longitudes
    :param grid: draw grid at specified intervals. Defaults to true
    :param save: save figures. Defaults to false
    :param path: filepath to save figure
    :return: azimuthal equidistant map centered on reference coordinates
    """
    m = Basemap(projection='aeqd', lat_0=lat_ref, lon_0=lon_ref)
    m.drawmapboundary(fill_color='white')
    # draw coasts and fill continents.
    m.drawcoastlines(linewidth=0.5)
    m.fillcontinents(color='lightgrey', lake_color='white')

    # 20 degree graticule
    if grid is True:
        m.drawparallels(np.arange(-80, 81, 20))
        m.drawmeridians(np.arange(-180, 180, 20))

    # draw a black dot at the center.
    #xpt, ypt = m(lon_ref, lat_ref)
    #m.plot([xpt], [ypt], 'ko')

    for x1, y1, c in zip(lons, lats, M):
        xpts, ypts = m(x1, y1)
        if float(c) >= 7.5:
            m.scatter(xpts, ypts, color='red', alpha=0.8, edgecolors='black', s=45, zorder=10)
        elif 6 <= float(c) < 7.5:
            m.scatter(xpts, ypts, color='yellow', alpha=0.8, edgecolors='black', s=30, zorder=5)
        else:
            m.scatter(xpts, ypts, color='green', alpha=0.8, edgecolors='black', s=15, zorder=0)

    # draw the title.
    plt.title('{}'.format(title))

    if save is True:
        name = date.today()
        plt.savefig(os.path.join(path, 'azeq_{}.pdf'.format(name)))

    plt.show()

def check_data(filepath):
    """

    :param filepath: filepath of coordinates
    :return: data exists, true. Does not exist, false
    """
    data = np.loadtxt(filepath)
    if len(data[0]) > 1:
        return True
    else:
        return False

def get_event_data(enter_dir, list2sort):
    """

    :param enter_dir: filepath of SAC files to plot events
    :return: event coordinates
    """
    evlats = []
    evlons = []
    M = []
    counter = 0

    if os.path.isdir(enter_dir):
        for path in os.listdir(enter_dir):
            if str(path[-3:]) == "sac":
                if path[-33:] in list2sort:
                    counter += 1
                else:
                    try:
                        st = read(os.path.join(enter_dir, path))
                        evlat = str(st[0].stats.sac.evla)
                        evlon = str(st[0].stats.sac.evlo)
                        mag = str(st[0].stats.sac.mag)
                        evlats = np.append(evlats, evlat)
                        evlons = np.append(evlons, evlon)
                        M = np.append(M, mag)
                        # suppress errors so code doesn't stop
                    except FileNotFoundError:
                        print("     error for file: ", path)
            else:
                print("     not a sac file")

    filename = input("Enter the name of the coordinate file that will be saved:")
    figname = str(filename) + ".txt"
    with open("{}".format(figname), 'w+') as f:
        writer = csv.writer(f, delimiter='\t')
        writer.writerows(zip(evlats, evlons, M))

    return evlats, evlons, M, filename, counter


# 0 if you don't have a file
filepath = 0
badfilepath = 0

# column number (0 - N columns)
lat_loc = 0
# column number (0 - N columns)
lon_loc = 1
# column number (0 - N columns)
mag_loc = 2

# reference location (e.g. middle station cluster)
ref_lat = 42.28     # Ann Arbor, MI
ref_lon = -83.78     # Ann Arbor, MI

if badfilepath != 0:
    ListToSort = bad_events(badfilepath)
else:
    ListToSort = []

if filepath == 0:
    title = input("Enter title of map: ")
    enter_dir = input("Enter the path of SAC events: ")
    evlats, evlons, M, filename, counter = get_event_data(enter_dir, ListToSort)
    print("Making map...")
    print("     ", counter, "bad events flagged")
    _fig0(title, ref_lat, ref_lon, evlats, evlons, M, enter_dir, grid=False, save=True)
else:
    loc = check_data(filepath)
    if loc is True:
        print("Data is in proper format")
        title = input("Enter title of map: ")
        enter_dir = input("Enter the path of working dir: ")
        print("Making map...")
        data = np.loadtxt(filepath)
        _fig0(title, ref_lat, ref_lon, data[:, lat_loc], data[:, lon_loc], data[:, mag_loc], enter_dir, grid=False, save=True)
    else:
        print("Could not make map, data is only 1 column")
```
