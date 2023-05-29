+++
title = "Quality control"
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
import time
import math
import numpy as np
import matplotlib as mpl
mpl.use('macosx')

# --------------------------
# RF QC code
# Currently, this code allows the user to select 5 bad traces at a time,
# writes the baz offsets to a .txt. Successive runs of the code reads the
# baz offsets .txt and plots the traces at a lower opacity, and marks the
# file in {station}_BAD_file.txt, and ALL_BAD_file.txt.
#
# For further analysis, the user can prompt code to look through
# {station}_BAD_file.txt or ALL_BAD_file.txt to omit bad files.
# --------------------------

station = 'K48A'

# Rounds to nearest 10
def roundup(x, n=10):
    """

    :param x: value
    :param n: integer to round to
    :return: rounded value
    """
    res = math.ceil(x / n) * n
    if (x % n < n/2) and (x % n > 0):
        res -= n
    return int(res)

# Display commands on figure
def tellme(s):
    """

    :param s: command to display
    :return: figure with command
    """
    print(s)
    plt.title(s, fontsize=16)
    plt.draw()

def _fig0(offset, scale_factor=1, line_opacity=1.0, fill_opacity=0.7, xlim=500):
    """

    :param offset: offset value
    :param scale_factor: scale trace up or down; defaults to 1
    :param line_opacity: opacity of trace line
    :param fill_opacity: opacity of positive and negative fill
    :param xlim: essentially, width of plot; defaults to 500
    :return: record section of traces
    """
    ax0.set_ylim([20, 0])
    ax0.set_xlim([0, xlim+10])
    ax0.set_xticks([])
    ax0.set_ylabel('time lag, s')
    ax0.minorticks_on()
    ax0.grid(which='major', axis='y', linewidth=0.5)
    fig.suptitle('{}'.format(station))
    ax0.plot(tr.data * scale_factor + offset, time, color='black', linewidth=0.8, alpha=line_opacity)
    ax0.fill_betweenx(time, tr.data * scale_factor + offset, offset, where=(tr.data * scale_factor + offset) > offset, color='r', alpha=fill_opacity)
    ax0.fill_betweenx(time, tr.data * scale_factor + offset, offset, where=(tr.data * scale_factor + offset) < offset, color='b', alpha=fill_opacity)

# Begin reading in files and sorting by baz
temp_array = []
filenames = []
totalRFs = 0
st = Stream()

for f in glob.iglob("/Users/madeleinetan/Research/arrays_downloaded_112422_master/all/TA.{}.*.sac".format(station)):
    st += read(f)
    filenames = np.append(filenames, f)

for tr in st:
    temp_array = np.append(temp_array, tr.stats.sac.baz)

idx = np.argsort(temp_array)
sorted_baz = temp_array[idx]

st1 = Stream()
SortedFiles = []
SORTED = []

count = 0
for i in filenames[idx]:
    xx = i[-33:]
    SortedFiles = np.append(SortedFiles, xx)
    st1 += read(i)
    count += 10

# Figure plotting parameters
tmin = st1[0].stats.sac.b
npts = st1[0].stats.sac.npts
dt = st1[0].stats.sac.delta
time = np.linspace(0, npts - 1, npts) * dt + tmin

fig = plt.figure()
grid = plt.GridSpec(5, 10)
ax0 = fig.add_subplot(grid[0:5, 0:10])

tellme('Plotting receiver functions...')

# Check if {station}_Bad.txt exist, if not create it
if not os.path.exists('/Users/madeleinetan/Research/arrays_downloaded_112422_master/{}_BAD.txt'.format(station)):
    with open('/Users/madeleinetan/Research/arrays_downloaded_112422_master/{}_BAD.txt'.format(station), 'w'): pass

# Plot traces; if offset == baz in bad event file, trace is plotted with lower opacity
offset = 10
badfilenames = []
for j, tr in enumerate(st1):
    with open('/Users/madeleinetan/Research/arrays_downloaded_112422_master/{}_BAD.txt'.format(station), 'r+') as file:
        dat = np.loadtxt('/Users/madeleinetan/Research/arrays_downloaded_112422_master/{}_BAD.txt'.format(station))
        if offset in dat:
            print("     1 bad trace marked:{}".format(tr.stats.sac.baz))
            badfilenames = np.append(badfilenames, SortedFiles[j])
            _fig0(offset, scale_factor=25, line_opacity=0.05, fill_opacity=0.05)
            offset += 10
            pass
        else:
            _fig0(offset, scale_factor=25, xlim=count)
            offset += 10
            totalRFs += 1

print("total satisfactory RFs = ", totalRFs)

# Select 5 bad events
done = False
while not done:
    BadPoints = []
    while len(BadPoints) < 5:
        tellme('Select 5 bad traces')
        BadPoints = np.asarray(plt.ginput(5, timeout=0, show_clicks=True))
        if len(BadPoints) < 5:
            tellme('Restarting QC...')

    # Plot a green dot on the RFs marked as bad
    RoundedBadPoints = np.around(np.array(BadPoints)) # whole number operation
    ph = plt.scatter(RoundedBadPoints[:, 0], RoundedBadPoints[:, 1], c='g', marker='.')

    tellme('Ready to save? Key input for yes, mouse click for no')

    done = plt.waitforbuttonpress()

    XBad = RoundedBadPoints[:, 0]


# Save bad events
    if done:
        XBads = []
        offset2index = []
        tellme('Initiating saving...')
        for n in XBad:
            XBads = np.append(XBads, float(roundup(n, 10)))

        for b in XBads:
            offset2index = np.append(offset2index, (int(b) / 10) - 1)

        offset2index = np.array(offset2index, dtype=int)
        index2save = SortedFiles[offset2index.T]

        # Save X,Y information of bad traces
        a_file = open('/Users/madeleinetan/Research/arrays_downloaded_112422_master/{}_BAD.txt'.format(station), "a")
        data = np.array([XBads]).T
        np.savetxt(a_file, data.astype(int), fmt='%i')
        a_file.close()

        # Save filename of bad traces at a station
        with open('/Users/madeleinetan/Research/arrays_downloaded_112422_master/{}_BAD_files.txt'.format(station), 'a') as f:
            for line in index2save.T:
                f.write(f"{line}\n")

        # Save filename of bad traces for all stations
        with open('/Users/madeleinetan/Research/arrays_downloaded_112422_master/ALL_BAD_files.txt', 'a') as f:
            for line in index2save.T:
                f.write(f"{line}\n")

        print(XBads)
```
