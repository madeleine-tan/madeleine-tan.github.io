+++
title = "Image scan, error propagation"
hideAuthor = true
hideDate = true
hideLastMod = true
+++

```python
# Creates a 3D surface plot or 3d histogram of crystal image scan

import numpy as np
import matplotlib.pyplot as plt
import matplotlib as mpl
mpl.use('macosx')

data_name = 'Image scan of test data'
data_array = np.loadtxt('./TBCL_FineScan1_copy.txt', skiprows=13)
A = data_array.T
length = int(np.sqrt(len(A)))
data = np.reshape(A, (-1, length))  # Makes into matrix
labels = np.linspace(-1,1,21) # Manually change this
labels = np.round(labels,2)

# For surface plot
nx, ny = length, length
x = range(nx)
y = range(ny)

f = plt.figure()
h = f.add_subplot(111, projection='3d')

X, Y = np.meshgrid(x, y)  # `plot_surface` expects `x` and `y` data to be 2D
surf = h.plot_surface(X, Y, data, cmap=plt.cm.hsv, edgecolors='k', lw=0.6)
h.set_zlabel('counts')
h.set_ylabel('Theta')
h.set_xlabel('Phi')
h.set_xticks(range(0, length), labels)
h.set_yticks(range(0, length), labels)
n = 4  # Keeps every 2nd label
[l.set_visible(False) for (i,l) in enumerate(h.xaxis.get_ticklabels()) if i % n != 0]
[l.set_visible(False) for (i,l) in enumerate(h.yaxis.get_ticklabels()) if i % n != 0]
f.colorbar(surf, shrink=0.5, aspect=5)
plt.title('{}'.format(data_name))
plt.show()

'''
# For 3D histogram plot

xpos = [range(matrix.shape[0])]
ypos = [range(matrix.shape[1])]
xpos, ypos = np.meshgrid(xpos, ypos)
zpos = np.zeros_like(xpos)
xpos = xpos.flatten('F')
ypos = ypos.flatten('F')
zpos = np.zeros_like(xpos)

dx = 0.5 * np.ones_like(zpos)
dy = dx.copy()
dz = matrix.flatten()

cmap = mpl.cm.get_cmap('jet') # Get desired colormap - you can change this!
max_height = np.max(dz)   # get range of colorbars so we can normalize
min_height = np.min(dz)
# scale each z to [0,1], and get their rgb values
rgba = [cmap((k-min_height)/max_height) for k in dz]

ax.bar3d(xpos, ypos, zpos, dx, dy, dz, color=rgba, zsort='average')
plt.show()
'''

# Error propagation code for RBS data
# Adapted from Joshua Cooper's IBA_error_propagation.m

import numpy as np
import pandas as pd
np.set_printoptions(precision=3)
pd.set_option("display.precision", 3)

# REFERENCE SAMPLE (this is 2019 TBCL)--------------------
N_a0 = 411
N_r0 = 26517
err_a0 = np.sqrt(N_a0)
err_r0 = np.sqrt(N_r0)
X0 = N_a0/N_r0
err_X0 = X0 * np.sqrt((err_a0/N_a0)**2 + (err_r0/N_r0)**2)

# SAMPLES--------------------
  # if there are multiple samples, separate values by a comma
N_aligned = np.array([486, 1396, 783])
N_random = np.array([22232, 22047, 21495])

# INITIALIZE MATRIX
sh_a = np.shape(N_aligned)
sh_r = np.shape(N_random)
sh_NdN = np.zeros(sh_a, )
sh_Xmin = np.zeros(sh_a, )
sh_NdN_err = np.zeros(sh_a, )
sh_err_align = np.zeros(sh_a, )
sh_err_rand = np.zeros(sh_a, )

matrix = (np.matrix([N_aligned, N_random, sh_Xmin, sh_err_align, sh_err_rand,
                     sh_NdN, sh_NdN_err])).T

# CALCULATIONS--------------------
for r in range(sh_a[0]):
  err_align = np.sqrt(matrix[r,0])
  matrix[r,3] = err_align

  err_rand = np.sqrt(matrix[r,1])
  matrix[r,4] = err_rand

  Xmin = matrix[r,0]/matrix[r,1]
  matrix[r,2] = Xmin

  err_Xmin = Xmin * np.sqrt((err_align/matrix[r,0])**2 +
                            (err_rand/(matrix[r,1]))**2)
  Xdif = Xmin - X0
  err_Xdif = np.sqrt((err_Xmin)**2 + (err_X0)**2)

  NdN = (Xmin-X0)/(1-X0)
  matrix[r,5] = NdN

  err_NdN = NdN * np.sqrt((err_Xdif/Xdif)**2 + (err_X0/(1-X0))**2)
  matrix[r,6] = err_NdN

# PRINT MATRIX--------------------
df = pd.DataFrame(matrix, columns=['Aligned counts', 'Random counts', 'Xmin',
                                   'Aligned error', 'Random error', 'Nd/N',
                                   'Nd/N error'])
df
```
