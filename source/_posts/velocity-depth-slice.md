---
title: velocity depth slice
date: 2018-09-23 00:33:07
tags: 
cover_image: examples/ex001/hello.jpg
---




```python
#!/usr/bin/env python
#!/usr/bin/env python
import pyGmt
import subprocess
import os
import numpy as np
"""
Plot along E-W
"""

R = "70/110/10/55"
vfile = "tibet_vp.xyzv"
figdir = "figs"
figsize = "12c"

r = R.split("/")
r0 = float(r[0])
r1 = float(r[1])
r2 = float(r[2])
r3 = float(r[3])

J = "B%f/%f/%f/%f/%s" % (0.5*(r0+r1), 0.5*(r2+r3), r2, r3, figsize)


for dep in range(50, 201, 25):

    datf = "figs/dv%s_p.xyv" % str(dep).zfill(3)

    psfile = "figs/dv%s_p.ps" % str(dep).zfill(3)
    print psfile


    gmt = pyGmt.Gmt()

    gmt.set("MAP_FRAME_TYPE", "plain")
    gmt.set("FORMAT_GEO_MAP", "ddd:mm:ssF")
    # gmt.set("MAP_DEGREE_SYMBOL", "none")

    gmt.cmd("pscoast", "-J%s -R%s -K -Bx10f5 -By10f5 -BWenS -W1/0.5p > %s" % (J, R, psfile))

    gmt.cmd("makecpt", "-Cjet -T7.5/8.5 -I > color.cpt")

    cmd = "cat %s | awk '{if($3==%f){print $1,$2,$4} }' > %s" % (vfile, dep, datf)
    subprocess.call(cmd, shell=True)


    gmt.cmd("xyz2grd", "%s -Gtemp.grd -I0.5/0.5 -R70/110/10/55" % (datf))
    gmt.cmd("grdsample", "temp.grd -Gout.grd -I0.1/0.1")
    gmt.cmd("grdimage", "out.grd -J -R%s -K -O -Ccolor.cpt >> %s" % (R, psfile))
    gmt.cmd("pscoast", "-J%s -R%s -K -O -W1/0.5p -Bwens >> %s" % (J, R, psfile))

    gmt.comment("plot tectoic line")
    gmt.cmd("psxy", "../../../moho/gmt/thrust.gmt  -R -J -W1.0p,black -Sf0.5+t+l -O -K >> %s" % (psfile))
    gmt.cmd("psxy", "../../../moho/gmt/sutures.gmt -R -J -W1.0p,black,- -O -K >> %s" % (psfile))
    gmt.cmd("psxy", "../../../moho/gmt/normal.gmt -R -J -W1.0p,black,- -O -K >> %s" % (psfile))
    # gmt.cmd("psxy", "../../moho/gmt/dextral.gmt -R -J -W1.0p,black,- -O -K >> %s" % (psfile))

    # label
    gmt.shell("echo 70.7 12 Depth=%.1fkm > text.d" % (dep))
    gmt.cmd("pstext", "text.d -J -R -K -O -F+f10p+jBL >> %s" % (psfile))

    # gmt.cmd("psxy", "CN-plate-neighbor.dat  -R -J -W3.0p,2/138/210 -Sf0.5+t+l -G2/138/210 -O -K >> %s" % (psfile))
    gmt.cmd("psxy", "../../../moho/CN-block-L1.dat -R -J -W1.0p,gray -O -K >> %s" % (psfile))
    gmt.cmd("psxy", "../../../moho/CN-block-L2.dat -R -J -W1.0p,gray -K -O >> %s" % (psfile))
    # gmt.cmd("psxy", "CN-faults.dat -R -J -W0.5p,black,- -K -O >> %s" % (psfile))

    # gmt.cmd("psxy", "moho_idw.xyz -J -R -K -O -Ccolor -St0.35c >> %s" % (psfile))
    gmt.cmd("psscale", "-DjCB+w7c/0.3c+o0/-2.0c+h -Ba0.5f0.1 -By+l'Vp [km/s]' -Ccolor -J -R -K -O >> %s " % (psfile))

    # plot station and events

    gmt.cmd("psxy", "-J -R -O -T >> %s" % psfile)
    gmt.cmd("psconvert", "-A -P -Tj -E720 %s" % psfile)
    gmt.cmd("psconvert", "-A -P -Tf %s" % psfile)

    gmt.execute()

    # os._exit(0)

```