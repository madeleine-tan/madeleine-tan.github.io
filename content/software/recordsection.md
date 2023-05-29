+++
title = "Record section"
weight = 1
hideAuthor = true
hideDate = true
hideLastMod = true
+++

```python
import matplotlib.pyplot as plt
from matplotlib import gridspec
from obspy import read, Stream
import numpy as np
import os
import glob
import matplotlib as mpl


def _fig0(data, offset, time, scale_factor=0.5, all=380):
    """
    Plot receiver functions as a function of back-azimuth (deg)

    Parameters:
        data: np.array
        scale_factor: scale trace up (f > 1), down (0 < f < 1). Default f=1
        offset: space traces out
        time: y-axis, calculated from sac header

    """
    ax0.set_ylim([20, 0])
    ax0.set_xlim([0, all + 2])
    ax0.set_xticks([])
    ax0.set_ylabel('time lag, s')
    ax0.minorticks_on()
    fig.suptitle('TA.{}'.format(station))
    ax0.plot(data * scale_factor + offset, time, color='black', linewidth=0.5)
    ax0.fill_betweenx(time, data * scale_factor + offset, offset, where=(data * scale_factor + offset) > offset,
                      color='r', alpha=0.7)
    ax0.fill_betweenx(time, data * scale_factor + offset, offset, where=(data * scale_factor + offset) < offset,
                      color='b', alpha=0.7)


def _fig1(data, time, scale_factor=1):
    """
    Plots receiver function stack [0, 20] seconds

    Parameters:
        data: np.array
        time: calculated using sac header
        scale_factor: scale trace up (f > 1), down (0 < f < 1). Default f=1

    """
    ax1.set_ylim([20, 0])
    ax1.grid(visible=True, which='major', axis='x', linewidth=0.5)
    ax1.set_yticks([])
    ax1.plot(data * scale_factor, time, color='black', linewidth=0.8)
    ax1.fill_betweenx(time, data * scale_factor, where=(data * scale_factor) > 0, color='r', alpha=1)
    ax1.fill_betweenx(time, data * scale_factor, where=(data * scale_factor) < 0, color='b', alpha=1)


rootdir = '/Users/madeleinetan/Research/arrays_downloaded_112422_master/'

text_file = open('/Users/madeleinetan/Research/arrays_downloaded_112422_master/ALL_BAD_files.txt', "r")
ListToSort = text_file.read().splitlines()
ListToSort = np.array(ListToSort)
text_file.close()

for j in os.listdir(rootdir):
    enter_dir = os.path.join(rootdir, j)
    if os.path.isdir(enter_dir):

        # initialize arrays
        all = 0
        temp_array = []
        filenames = []
        st = Stream()
        suffix = str('TA.K48A.*.sac')

        for f in glob.iglob(os.path.join(enter_dir, suffix)):
            if f[-33:] in ListToSort:
                pass
            else:
                st += read(f)
                filenames = np.append(filenames, f)
                all += 1

        for tr in st:
            temp_array = np.append(temp_array, tr.stats.sac.baz)

        temp_array = np.array(temp_array, dtype=int)
        idx = np.argsort(temp_array)

        st1 = Stream()
        filenames = np.array(filenames)

        for i in filenames[idx]:
            st1 += read(i)

        # set parameters using first trace in stream
        station = str(st1[0].stats.station)
        tmin = st1[0].stats.sac.b
        npts = st1[0].stats.sac.npts
        dt = st1[0].stats.sac.delta
        time = np.linspace(0, npts - 1, npts) * dt + tmin

        # set parameters using first trace in stream
        fig = plt.figure()
        grid = plt.GridSpec(6, 8, hspace=0.2, wspace=0.2)
        ax0 = fig.add_subplot(grid[0:6, 0:6])  # rfs
        ax1 = fig.add_subplot(grid[0:6, 6])  # stack

        offset = 1
        for j, tr in enumerate(st1):
            # plot RF trace
            _fig0(tr.data, offset, time, 7, all)

            offset += 1

        # to save stack
        # stack = st1.stack()
        # save stack
        # stack.write(‘{}_stack.sac', format='SAC')

        stack2 = np.sum([tr.data for tr in st1], axis=0)
        _fig1(stack2, time)

        # save figures as pdf
        plt.savefig(os.path.join(enter_dir, str('{}_stack.png'.format(station))), dpi=600)

        print("Total RFs used: ", all)
        print("success for:", enter_dir)
        plt.show()
```
