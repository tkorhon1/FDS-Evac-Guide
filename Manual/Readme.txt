FDS+Evac Readme File

Timo Korhonen
VTT Technical Research Centre of Finland

=======================================
User Input Changes, Evac 2.6.0 vs 2.5.1 (FDS 6.7.5)
=======================================

Main user input changes made to evacuation inputs.  The old input
files do not work anymore.  The fire+evacuation coupling strategy is
totally changed.  Now the evacuation and fire meshes are separated to
their own runs (same fds.exe is used, but the new keywords present at
the MISC namelist will dictate what is calculated).

NOTE: Just the fire + evaucuation coupling input (MISC namelist) is
changed, the other "old" evacuation inputs are unchanged ==> You need
just to change the MISC namelist of your old input file.

The MISC namelist has now two new logical keywords:
EVACUATION_INITIALIZATION (default F)
EVACUATION_WRITE_FED (default F)
Also the "old" keyword EVACUATION_MC_MODE (default F) is used in the
new fire+evacuation strategy to say, which part of the calculation is run.

Fire and evacuation meshes separated, a complete fire+evacuation
simulation is done in three phases:
 
 1) EVACUATION_INITIALIZATION=T: => CHID_evac.eff and CHID_evac.xyz
    data files are written to the hard drive (for later use).  Alse an
    evacuation drill simulation is done (no fire information is used).
    HINT: Check that the drill simulation seems to be fine before you
    do the fire calculation (fire => much CPUs => you do not want to
    rerun the fire part, if you need to change the evacuation geometry
    inputs)

 2) EVACUATION_WRITE_FED=T: fire calculation reads CHID_evac.xyz and
    writes CHID_evac.fed file.  This is an ordinary fire calculation,
    but it read the (x,y,z) information of the evacuation meshes,
    where to save smoke etc. information to the disk.  And writes the
    FED file that is to be used in the phase 3, where evacuation using
    smoke information is done.
    NOTE: If no chid_evac.xyz file not found => stop (no accidental
    long runs) 

 3) EVACUATION_MC_MODE=T: An evacuation calculation that reads
    CHID_evac.eff and CHID_evac.fed from the disk.
    NOTE: If no CHID_evac.eff found => stop
          If EVACUATION_DRILL=F and no CHID_evac.fed found => stop

HINT: How to transfer an old FDS+Evac input file to the new strategy?
   a) Copy the old CHID.fds => CHID_1.fds, CHID_2.fds, CHID_3.fds
   b) Edit (text editor):
      CHID_1.fds: &MISC EVACUATION_INITIALIZATION=.TRUE. /
      CHID_2.fds: &MISC EVACUATION_WRITE_FED=.TRUE. /
      CHID_3.fds: &MISC EVACUATION_MC_MODE=T=.TRUE. /
   c) Run CHID_1.fds
      Copy CHID_1_evac.xyz => CHID_2_evac.xyz
      Copy CHID_1_evac.eff => CHID_3_evac.eff
      Run CHID_2.fds
      Copy CHID_2_evac.fed => CHID_3_evac.fed
      Run CHID_3.fds

=======================================
User Input Changes, Evac 2.5.1 vs 2.5.0 (FDS 6.3.2)
=======================================

No changes, just some bug fixes etc.

=======================================
User Input Changes, Evac 2.5.0 vs 2.4.1 (FDS 6.1.2)
=======================================

*) Some bug fixes, the user input is not changed.

=======================================
User Input Changes, Evac 2.4.0 vs 2.3.1 (FDS 6.0.0)
=======================================

*) The door algorithm uses different height for "see or not" than the
 main evacuation mesh that contains the obstacles for movement.  The
 "see or not" mesh is at the height (z_smoke) defined by
 HUMAN_SMOKE_HEIGHT and the EVAC_Z_OFFSET.  The z1 = z_smoke -
 EVAC_DELTA_SEE and z2 = z_smoke + EVAC_DELTA_SEE are used for the z
 range of this see or not mesh.  The names of these additional meshes
 are like ID='Emesh_MESH_ID', where MESH_ID is the name of the
 corresponding main evacuation mesh.  One may have to use these mesh
 IDs to specify additional holes and obsts for these see-or-not
 meshes, but usually just specifying EVACUATION=.TRUE. is enough.

*) No need to specify STRS meshes. The XB size hole is generated
 automatically and also the core OBST is generated automatically.
 The automatically generated mesh have ID='Emesh_STRS_ID', where
 STRS_ID is the ID of the STRS namelist. This ID may be used to put
 some additional obstacles to the stairs, if needed.  Usually just
 specifying EVACUATION=.TRUE. is enough.

*) No need to specify the additional door flow fields.  No need to
 define "outflow" vents.  No need to define additional obsts for the
 "outflow" vents.  The user should only define the main evacuation
 meshes and the EXITs and DOORs, all the above other things are
 generated automatically.

*) If CORR is used and its TO_NODE is an "dummy" EXIT then the exit
 should have COUNT_ONLY=.TRUE. so that there will be no mesh (and flow
 field calculation) for this dummy exit.  Also SHOW=.FALSE. might be
 good, i.e., do not show the dummy exit in Smokeview.

*) Added logical keyword LOCKED_WHEN_CLOSED (default is false) to the
 EXIT/DOOR namelists.  If it is set true then the agents stop at the
 door line if the door/exit is closed.  In normal mode the agents can
 go through closed doors, but they do not use closed doors in the door
 selection algorithm.

*) Added logical keyword TARGET_WHEN_CLOSED (default is false) to the
 EXIT/DOOR namelists.  If it is set true then the door/exit is
 included in the door selection algorithm even if it is closed.  The
 agents will go towards closed doors and will go through the closed
 doors if LOCKED_WHEN_CLOSED is not set to true.

*) Added a logical keyword to (some) PERS namelist: EVAC_FDS6 is by
 default .FALSE. If it is set to .TRUE. then the agents aim straight
 towards the doors that they can see.  Firts they aim towards XYZ and
 when they are close enough then they start to aim towards XB.  "Aim
 towards XB or XYZ" does not mean straght to that point. The agents
 aim between the door posts (minus 30 cm for the body size effect) and
 the mid point of the door (XYZ is a point but the door width is also
 used here to place "door posts" for this point).  If the agents are
 within the door posts of the door (say, door has IOR=+1, and
 y1=4.0m and y2=6.0m, then if the agent has y=4.3m to 5.7m) then the
 agent will (try to) move directly along the +x direction.  If the
 agent density is high around the agents (more than 2 p/m2) then the
 guiding flow field of the target door is used.  Same is true when
 no XB nor XYZ are visible.  The left and right side densities are
 checked separately (close to a wall or at the outer boundary of a
 agent crowd). If "straight towards door" would mean turning towards
 a high left (or right) density then the flow field is used instead.

*) If EVAC_FDS6=.TRUE. is given then the user should not give any
 additional evacuation flow fields nor vents.  There are two basic
 levels in the evacuation meshes.  One is the main evacuation mesh
 location at the z-level given by the XB on the mesh line.  The OBSTs
 (and HOLEs) on this mesh will be used to the movement of the agents.
 These are also used to calculate the guiding flow fields towards
 different exit doors.  The other z-level is defined with the
 HUMAN_SMOKE_HEIGHT parameter (measured from the "floor level").  This
 gives its mid z-position, the z-range is plus/minus EVAC_DELTA_SEE
 (default 0.29 m), which can be given on (some) PERS namelist.  This
 new level is used with the door selection algorithm to decide what
 the agents are able to see.  Also the smoke information is at this
 level.

=======================================
User Input Changes, Evac 2.3.1 vs 2.3.0
=======================================

*) Added herding behaviour for agents.
 EVAC-namelist:
 AGENT_TYPE: Default is 2
    1 rational agents (visible doors and known doors equal)
    2 known doors first, then the visible ones (like in the manual)
    3 herding (choose the door that the others are using if no known doors given)
 PERS-namelist: I_HERDING_TYPE=0 (default)
   0: default herding (follow the default flow field if no door found)
   1: keep the first choice (follow the default flow field if no door found)
   2: do not move, if no door found
   3: do not move, if no door found + keep the first choice
   (keep the first choice: If the agent does know/see any door then it tries
    to check the other agents.  If it finds this way a door, then it does not
    try anymore to find any other door. The other mode of operation is that
    the agent constantly sees what the others are doing around and changes
    its mind accordingly, if no known or visible door available.)

=======================================
User Input Changes, Evac 2.3.0 vs 2.2.2
=======================================

*) DOOR namelists: keep_xy=.true. is the default now.

=======================================
User Input Changes, Evac 2.2.2 vs 2.2.1
=======================================

*) FED_DOOR_CRIT = -100.0 is the default, i.e., door selection
   algorithm is classifying the doors according to the smoke density
   (visibility criteria).  It used to be a FED criteria.

=======================================
User Input Changes, Evac 2.2.1 vs 2.2.0
=======================================

*) SVN version 6127 or later: Added logical keywords NO_EVACUATION
   (default false) and EVACUATION_DRILL (default false) to the MISC
   namelist.  Now the user do not have to "comment out" the ampersands
   of the fire or evacuation mesh lines, if he/she wants just to do a
   plain fire or plain evacuation calculation.  The effect of the
   EVACUATION_MC_MODE (default false) is changed a little bit.  If it
   is set true then no fire meshes are used and the file is tried to
   read in and if EVACUATION_DRILL is false (the default) then also
   the FED file is read in.  In short: EVACUATION_DRILL
   =.TRUE. discards all fire information and all fire meshes.

*) Added Tpre and  Tdet distributions on the EVAC namelists.
   They are overriding the ones given on the PERS namelists.
   If no PRE_EVAC_DIST or/and DET_EVAC_DIST is/are given on EVAC
   namelist, the ones from the PERS nameelist referred by PERS_ID
   are used. 

*) Changed some default parameters of the collision avoidance
   model. These changes do not affect "normal evacuation", where
   there are no counterflows. With these changes the results are 
   closer to the experimental papers:
   - M. Isobe, T. Adachi, T. Nagatani, "Experiment and simulation of
     pedestrian counter flow", Physica A 336, p. 638-650, 2004
   - T. Kretz, A. Gruenebohm, M. Kaufman, F. Mazur, M. Schreckenberg,
     "Experimental study of pedestrian counterflow in a corridor",
     J. of Stat. Mechanics: Theory and Experiment, P10001, 2006 

 v2.2.0 CONST_DF=0.5 !CF: prefer agents going in the same direction
 v2.2.0 FAC_NOCF=0.5 !CF: prefer v0, if no counterflow
 v2.2.1 CONST_DF=2.0 !CF: prefer agents going in the same direction
 v2.2.1 FAC_NOCF=2.0 !CF: prefer v0, if no counterflow

=======================================
User Input Changes, Evac 2.2.0 vs 2.1.2
=======================================

*) STRS namelist for staircases is working, it should be added to the
 manual.

*) Defaults are changed:
   - lam_non_sp = 0.3 (used to be 0.5)
   - Counterflow algorithm is used as default (TAU_CHANGE_V0 = 0.1)
   - Better waiting time for the door selection algorithm is used as
     default (FAC_DOOR_QUEUE = 1.3, FAC_DOOR_WAIT = 0.9)
   - DOOR: EXIT_SIGN = .TRUE. by default, if TO_NODE is given.  If
     TO_NODE is not given (or = 'null'), then this door is taken as a
     'dummy' door, i.e., one way door and EXIT_SIGN = .FALSE. always
     for these kind of doors.

*) How things are shown in Smokeview: Now the evacuation mesh obsts
 are shown as outlines (default), if there are both fire and
 evacuation meshes present in the calculation.  If there are just
 evacuation meshes then the obsts are shown as solid.

=======================================
User Input Changes, Evac 2.1.2 vs 2.1.1
=======================================

*) Output change: CHID_evac.csv header format changed.  The new format
   is the same as for the CHID_devc.csv file.  Two header rows
   followed by the data, first column is the time.

*) EXIT/DOOR/ENTR: SHOW=.TRUE. or .FALSE., Smokeview shows or does not
   show the objects as "devices".  Note, that COUNT_ONLY=.TRUE. exits
   are no shown at all.  Also HEIGHT can be given, it is the height of
   the door/exit/entr shown in Smokeview. (needs a new devices.svo
   file)

*) The SLCF should have also EVACUATION=.TRUE. if the guiding flow
   fields for evacuation are wanted to be plotted.

*) The log-normal distribution has two more parameters, now it can be
   truncated at the high end side and shifted. The parameters are:
   ave,   sigma of ln(x), high end cutoff, shift of x
   _MEAn, _PARA,          _HIGH,            _PARA2

=======================================
User Input Changes, Evac 2.1.1 vs 2.1.0
=======================================

*) "&ENTR" namelist: The user can give the maximum maximum number of
   agents to be introduced in the calculation by an entry by giving
   the keyword MAX_HUMANS.

*) "&MISC" namelist: EVACUATION_MC_MODE=.FALSE. by default, i.e.,
   CHID_evac.eff file is (re)calculated.  If true then eff file is
   tried to read in.  Fire data file (CHID_evac.fed) is always used if
   found on the disk.

*) The namelists EXIT, DOOR, and ENTR have their XB fitted to the mesh
   cells just like, e.g., VENT namelist.

*) Thicken evacuation mesh obstructions in the z direction is now done
   always.  This was done in the version 2.0.1 but not in 2.1.0.  This
   may bring about some changes to the old input files but these
   should not break any working v2.1.0 input file.


==========================
Latest changes, Evac 2.4.0
==========================

12.11.2010 No need to specify the additional door flow fields.  No
     need to define "outflow" vents.  No need to define additional
     obsts for the "outflow" vents.  The user should only define the
     main evacuation meshes and the EXITs and DOORs, all the above
     other things are generated automatically.

12.11.2010 If the door/exit is seen (XY and/or XYZ) the agents do not
      (necessarily) follow the flow field.  They might aim directly
      towards the door.  If both XYZ and XB is seen then the agents
      first aim towards XYZ and when they are close to then they start
      to aim XB.  Added a logical keyword to (some) PERS namelist:
      EVAC_FDS6 is by default .FALSE. If it is set to .TRUE. then the
      agents aim straight towards the doors that they can see.  First
      they aim towards XYZ and when they are close enough then they
      start to aim towards XB.  "Aim towards XB or XYZ" does not mean
      straght to that point. The agents aim between the door posts
      (minus 30 cm for the body size effect) and the mid point of the
      door (XYZ is a point but the door width is also used here to
      place "door posts" for this point).  If the agents are within
      the door posts of the door (say, door hax IOR=+1, and y1=4.0m
      and y2=6.0m, then if the agent has y=4.3m to 5.7m) then the
      agent will (try to) move directly along the +x direction.  If
      the agent density is high around the agents (more than 2 p/m2)
      then the guiding flow field of the target door is used.  Same is
      true when no XB nor XYZ are visible.  The left and right side
      densities are checked separately (close to a wall or at the
      outer boundary of a agent crowd). If "straight towards door"
      would mean turning towards a high left (or right) density then
      the flow field is used instead.

15.11.2010 The visibility is checked at different height than the
      obstructions for movement.

15.11.2010 The doors and exits have the option that when they are
      closed they do not let anybody throught them (in the check doors
      and exits routines the agents are stopped at the XB line).

29.11.2010 STRS defines automatically the mesh and makes it empty bu a
      hole and puts the core obst. Still k=1.

==========================
Latest changes, Evac 2.3.1
==========================

21.9.2010: Added herding behaviour for agents.  If an agent is herding
     type or an agent can not find any door then it uses its neighbors
     to find a door.  If there are no nearest neighbors that know a
     door then it tries to find (the closest) agent that knows a door
     and uses this information.

21.9.2010: Uses L1-norm for non-visible doors.
     Note also that for smokey doors the visible and non-visible ones
     are in the same gategory.

==========================
Latest changes, Evac 2.3.0
==========================

1.7.2010: Now tau is reduced on the "inclines", because the magnitude
    of the motive force is v0/tau and the motive force strength should
    not be changed on the inclines. Tau is decreased by the
    v0_incline/v0 ratio (minimum tau=0.1s)
 

28.6.2010: Corrected the STRS algorithm, now it finds better the doors
     and also strs=>door=>somewhere works better, now it is checked
     that there is empty space, before there was a bug here.

28.6.2010: keep_xy=.true. is now the default for doors. It is nice
     that this is done for STRS by default.

==========================
Latest changes, Evac 2.2.2
==========================

5.5.2010: FED_DOOR_CRIT = -100.0 as default, i.e., door selection
    algorithm is using visibility. The FED index is not anymore
    supported in the future versions of door (or route) selection
    algorithms.  Visibility: Can you see the door? Is there some smoke
    so you would like "no-smoke" door better.  Radiation levels:
    Choose direction, where the levels go down (if the levels are
    "large").  Temperature: Choose direction, where the levels go down
    (if the levels are "large").  The FED index could have an effect
    on the "stress level" of the agent.  Evac 2.2.2: Just the
    visibility is implemented.

==========================
Latest changes, Evac 2.2.0
==========================

28.9.2009: The STRS is now working and have door selection algorithm
     working at the final destination landing.

28.9.2009: If fire+evac then evacuation mesh obsts as outlines by
     default. If transparency (evac_surf_default line) is < 0.99999
     then "solid" transparent blocks for evacuation.

14.8.2009: Door queueing algorithm is the default.

14.8.2009: Counterflow algorithm is working and it is now the default.

14.8.2009: Added more user input checks

==========================
Latest changes, Evac 2.1.2
==========================

17.3.2009: The Fortran pointer definitions are taken away from the
     EVAC module definitions and put inside the subroutines.

17.3.2009: The door selection algorithm includes also the queueing time.

17.3.2009: The speed dependent random force treated better.

16.2.2009: Agent-Wall social forces are calculated differently than in
     the previous versions.  Now these forces are calculated similarly
     as the agent-agent social forces, i.e., closest circle is used.
     Previously the large body radius (the large body circle) was
     used.  This will change the flows a little bit, because the
     agents can be closer to the walls.

==========================
Latest changes, Evac 2.1.1
==========================

10.2.2009: EVACUATION_MC_MODE=.FALSE. by default (MISC-line), i.e.,
     eff file is (re)calculated.  If true then eff file is tried to
     read in.

10.2.2009: Bug fix: Thicken evacuation mesh obstructions in the z direction.

25.1.2009: No "target" columns for count_only exits (CHID_evac.eff).

21.1.2009: The EFF file read error is dealed better.  The calculation
    is not stopped, just a warning message is printed and the EFF
    file is recalculated (and overwritten).

19.1.2009: The namelists EXIT, DOOR, and ENTR have their XB fitted to
    the mesh cells just like, e.g., VENT namelist.

9.1.2009: The prefered walking direction field v0 is interpolated
    differently than in v2.1.0.  Old version used function AFILL,
    which used the closest four velocity (x,y directions, z is not
    needed for 2-dim case) values.  There was a problem if there was
    thin walls.  Then some of these four values used to interpolate
    the velocity might be on the other side of the thin wall, i.e.,
    interpolation used the values at the cell, where the agent is, and
    the values form an adjacent cell, but there was no check for
    walls.  The present version uses just the values in the current
    cell to interpolate the direction field.  (Note: the u-velocity is
    given at the cell faces -1,+1, the v-velocity is given at the cell
    faces -2,+2, see the FDS Tech. Guide)

    This modification will change the FDS+Evac results slightly,
    but one can use the old evacuation flow fields (CHID_evac.eff).
    These fields are not modified, but they are used differently,
    i.e., interpolation is done differently.

9.1.2009: v2.1.1 uses less memory than v2.1.0, because radiation
    related arrays are not allocated for the solid state boundaries
    for the evacuation meshes.  For the example case
    "evac_example2a.fds" the savings are about 18 % less RAM.

=======================================
User Input Changes, Evac 2.1.0 vs 2.0.1
=======================================

*) Exit doors (with an "outflow" VENT) can now be defined directly on
   a wall, i.e., no need to make a small hole in the wall to get the
   forces from the door casings (See Evac version 2.0.0 manual, section
   1.2, Figure 1)

*) The default THICKEN=.FALSE. for OBST-lines in the version 2.1.0.
   This version can use zero-thick obstructions correctly. Older
   versions were always using at least one grid cell thick
   obstructions.

*) COLOR_INDEX, AVATAR_RGB, AVATAR_COLOR, COLOR, RGB can be used to
   define the colors of the avatars in Smokeview.  See the latest
   version of the FDS+Evac manual for details.

==========================
Latest changes, Evac 2.1.0
==========================

15.10.2008: New dt calculation for the agent movement algorithm.
      Minimum movement distance in one dt is 0.0001 m or larger.
      EVAC_DT_MIN=0.001s and EVAC_DT_MAX=0.01s by default.  Dt is
      decreased as much that the agents do not move more than 0.2
      times the agent-agent (or agent-wall) distance.  The agent-agent
      minimun distance takes care about the velocities of the agents
      by using scalar product of the unit velocity vectors to scale the
      distance.  Note that increasing the EVAC_DT_MAX makes the
      calculation to run faster but it can not be larger than the
      EVAC_DT_STEADY_STATE.

8.10.2008: Now the random force is scaled better for inclines.  Now
   the local target speed (on the xy-projection) is used.

==========================
Latest changes, Evac 2.0.1
==========================

19.8.2008: Now AVATAR_COLOR, AVATAR_RGB, COLOR, and RGB can be used
    for evacuation input.  Do not use QUANTITY and COLOR_INDEX
    anymore. 

19.8.2008 Now T_BEGIN works on the TIME namelists for evacuation.

4.7.2008: Now EVACUATION=.FALSE. is default for all VENTs so there is
    no need anymore to specify this for the fire calculation VENTs.

4.7.2008: HOLEs are read in better.  Now the EVACUATION=.FALSE. or
    .TRUE. logic is the same as for OBSTs, i.e., by default (no
    EVACUATION keyword) holes are going both to the fire and
    evacuation meshes.

2.7.2008: main_mpi.f90 supports now fire+evacuation calculation.
    Evacuation meshes should be defined after the fire meshes in the
    input file.  Evacuation meshes are treated as a single process.
    Below is an exampe config.txt file for MPICH2+WindowsXP for the
    FDS+Evac User's Guide example case 2 (also at the FDS+Evac web
    page).  It has two fire meshes, two main evacuation meshes and
    four additional evacuation flow field meshes, so this adds up to
    three processes (2 fire + 1 evac).  Below the first fire mesh goes
    to the machine "fire1" and the second to "fire2" and the
    evacuation part is calculated in "fire1".  This way the CPU
    intensive fire calculations are in different CPUs.

      exe \\fire1\MPI_Tests\fds5_mpi.exe evac_example2a.fds
      dir \\fire1\MPI_Tests
      hosts
      fire1 1
      fire2 1
      fire1 1

    And below is the output to the stderr channel:

      Process   2 of   2 is running on fire1.vtt.fi
      Process   0 of   2 is running on fire1.vtt.fi
      Process   1 of   2 is running on fire2.vtt.fi
      Mesh   1 is assigned to Process   0
      Mesh   2 is assigned to Process   1
      Mesh   3 is assigned to Process   2
      Mesh   4 is assigned to Process   2
      Mesh   5 is assigned to Process   2
      Mesh   6 is assigned to Process   2
      Mesh   7 is assigned to Process   2
      Mesh   8 is assigned to Process   2
