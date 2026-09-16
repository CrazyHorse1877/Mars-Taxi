Space taxi
An interactive simulation of a shuttle running between Earth and Mars, planned with real orbital mechanics. It is packaged as a single HTML file with no build step and no dependencies.

The taxi waits on each planet until Earth and Mars line up, flies the best trajectory it can find under your rules, and returns at the next good launch window. Every trip is designed with a Lambert solver, the same technique mission planners use, and every candidate trajectory is shown on a porkchop plot you can click to choose your own.

Quick start
Open space-taxi.html in any modern browser (Chrome, Firefox, Safari, Edge).

The page works offline. It loads two typefaces (Cormorant Garamond and IBM Plex Sans) from Google Fonts when a connection is available and falls back to system fonts otherwise.

To serve it locally instead:

python3 -m http.server 8000
# then visit http://localhost:8000/space-taxi.html
Using the simulation
The mission cycle
The simulation starts on today's date with the taxi parked on Earth, and repeats this cycle indefinitely:

Wait on Earth until the planned launch date.
Fly to Mars on the chosen transfer orbit.
Arrive at Mars. The taxi immediately plans the return, searching launch dates starting after the minimum stay.
Wait on Mars until the return window.
Fly back to Earth, then plan the next outbound trip the same way.
With the default rules, the first launch is 31 October 2026, which falls in the real 2026 Earth–Mars launch window. A full round trip takes around four to five years, because after arriving at Mars the taxi usually has to wait about a year for Earth to come back into position.

The map
Planet orbits are drawn for Mercury, Venus, Earth and Mars at the current date.
The dashed arc is the planned route. Once the taxi launches, the flown part turns solid.
Hollow dotted rings show where the departure planet will be at launch and where the destination planet will be on arrival. They show why launches aim at where the target planet will be, not where it is now.
While the taxi is parked, dotted lines and an arc show the current angle between the planets. The status line compares it with the textbook Hohmann launch angle: Mars 44° ahead of Earth for the trip out, and Earth 75° behind Mars for the trip home.
Recent trips remain as faint paths.
Drag to pan, scroll or pinch to zoom.
The porkchop plot
The chart below the map shows every trajectory the planner evaluated for the current or next trip:

The horizontal axis is the launch date, covering 800 days from the earliest allowed launch.
The vertical axis is the flight time, from 90 to 450 days.
Colour shows the total Δv needed, from dark blue (cheapest) through green and yellow to orange. Trips more than 6 km/s above the cheapest option, and trajectories that are impractical, are left blank.
Faded cells break your current rules: flights longer than the limit, or trips over the fuel budget in "Fly fastest" mode.
The ring marks the trip the taxi will fly, the dashed line is the flight time limit, and the vertical line is the current date moving toward launch.
Hover over any point to see its launch date, flight time, arrival date and total Δv. Click a point to fly that trajectory instead. You can only change a trip that hasn't launched yet, and only to a launch date that hasn't passed.

Controls
Pause / Play stops and resumes the clock.
Skip to launch or Skip to arrival jumps to just before the next event.
Start over resets to today with a fresh service record.
Speed runs from 1 to 200 days per real second.
Taxi rules
Use least fuel picks the trajectory with the lowest total Δv.
Fly fastest picks the shortest flight whose total Δv fits the fuel budget per trip (5.5–20 km/s). If nothing fits, the taxi flies the cheapest trip and the panel says so.
Longest allowed flight (120–450 days) rules out slower trajectories.
Shortest stay on Mars and Shortest stay on Earth (0–700 days) set how long the taxi must stay before its next launch.
Changes to the flight limit, budget or mode immediately re-plan a trip that hasn't launched yet. Stay lengths apply from the next arrival.

Panels
Next trip / In flight lists launch and arrival dates, flight time, both engine burns, total Δv, and the speed relative to each planet on departure and arrival.
Service record counts trips, round trips, days in flight and total Δv used.
Log records launches, arrivals, upcoming windows and your changes.
How it works
Units and planet positions
Heliocentric calculations use astronomical units, years and G·M(Sun) = 4π². Speeds are converted to km/s for display (1 AU/year = 4.74047 km/s).

Planet positions and velocities come from JPL's approximate mean Keplerian elements for the J2000 epoch, advanced to each date with their rates of change per century. Kepler's equation is solved with Newton's method. The model is 2D: all orbits lie in the ecliptic plane and inclinations are ignored. The Earth entry uses the Earth–Moon barycentre.

Lambert's problem
Given two positions and the time to travel between them, Lambert's problem asks what orbit connects them. The solver uses the universal-variable formulation (Bate, Mueller & White; Curtis) with Stumpff functions, and finds the variable z by bisection, which is robust because flight time increases steadily with z for single-revolution transfers. It returns the spacecraft's velocity at departure and arrival. Only prograde, single-revolution transfers are considered, and trajectories that would escape the Sun are discarded.

Fuel cost
The speed relative to each planet on departure and arrival (v∞, the hyperbolic excess speed) is found by subtracting the planet's own velocity. Each burn is computed from a circular parking orbit using the Oberth-effect formula:

Δv = sqrt(v∞² + 2μ/r) − sqrt(μ/r)
The parking orbits are 400 km above Earth and 300 km above Mars. The total Δv for a trip is the departure burn plus the capture burn. Landing, launch from the surface and aerobraking are not included.

Planning a trip
For each trip the planner:

Evaluates a grid of launch dates every 4 days over 800 days, and flight times every 5 days from 90 to 450 days, about 14,600 Lambert solutions.
Picks the best grid point under the current rules.
Refines the choice with a finer search (half-day and one-day steps) around that point.
Planning takes a few hundred milliseconds or less. The chosen trajectory is converted to orbital elements and the taxi's position is computed from Kepler's equation at every frame, so its path is exact and lands precisely on the destination planet (tested to within 1e-13 AU).

Sample schedule
With the default rules (least fuel, flights up to 400 days, 30-day minimum stays), starting 17 September 2026:

Trip	Launch	Flight	Arrival	Total Δv
Earth → Mars	31 Oct 2026	311 days	7 Sep 2027	5.62 km/s
Mars → Earth	30 Aug 2028	316 days	12 Jul 2029	5.56 km/s
Earth → Mars	20 Jan 2031	257 days	4 Oct 2031	5.84 km/s
Mars → Earth	20 Jan 2033	244 days	21 Sep 2033	5.55 km/s
Switching to "Fly fastest" with a 9 km/s budget moves the first launch to 17 December 2026 and cuts the flight to 168 days, using 8.96 km/s.

<img width="983" height="610" alt="image" src="https://github.com/user-attachments/assets/5e6054cd-57bd-4f6a-93ce-6f7e1fe0a8e3" />

Project structure
space-taxi.html          The entire application: markup, styles and script
space-taxi-README.md     This file
Customising
The script is split into an orbital core and the app:

PLANETS holds the orbital elements and colours.
PARK sets the parking orbit radius and gravitational parameter for each planet.
G_DEP_STEP, G_DEP_SPAN, G_TOF_MIN, G_TOF_MAX and G_TOF_STEP set the porkchop search grid.
planLeg() contains the choice logic; better() inside it defines what "best" means for each mode.
STOPS and RANGE set the porkchop colour scale.
Adding another destination, such as Venus, needs an entry in PARK and a change to which planets the taxi alternates between.

Limitations
Orbits are 2D, so plane-change costs are ignored. Real porkchop plots have a ridge of very expensive trajectories near 180° transfers that doesn't appear here.
Only single-revolution, prograde transfers are searched.
Planets' gravity only matters at the burns (patched conics); there are no gravity assists or mid-course corrections.
Planet positions use mean elements, accurate to within a few degrees roughly between 1800 and 2050.
Further reading
R. R. Bate, D. D. Mueller and J. E. White, Fundamentals of Astrodynamics, Dover, 1971.
H. D. Curtis, Orbital Mechanics for Engineering Students, Butterworth-Heinemann (Lambert's problem, chapter 5).
E. M. Standish, "Keplerian Elements for Approximate Positions of the Major Planets," JPL Solar System Dynamics.
NASA, "Mars Exploration: Launch Windows" and the Mars Design Reference Architecture 5.0 for real mission Δv figures.Space taxi
An interactive simulation of a shuttle running between Earth and Mars, planned with real orbital mechanics. It is packaged as a single HTML file with no build step and no dependencies.
The taxi waits on each planet until Earth and Mars line up, flies the best trajectory it can find under your rules, and returns at the next good launch window. Every trip is designed with a Lambert solver, the same technique mission planners use, and every candidate trajectory is shown on a porkchop plot you can click to choose your own.

<img width="984" height="721" alt="image" src="https://github.com/user-attachments/assets/8ca54a97-24e7-48b3-8948-350fa4e18f35" />

Quick start
Open `space-taxi.html` in any modern browser (Chrome, Firefox, Safari, Edge).
The page works offline. It loads two typefaces (Cormorant Garamond and IBM Plex Sans) from Google Fonts when a connection is available and falls back to system fonts otherwise.
To serve it locally instead:
```bash
python3 -m http.server 8000
# then visit http://localhost:8000/space-taxi.html
```
Using the simulation
The mission cycle
The simulation starts on today's date with the taxi parked on Earth, and repeats this cycle indefinitely:
Wait on Earth until the planned launch date.
Fly to Mars on the chosen transfer orbit.
Arrive at Mars. The taxi immediately plans the return, searching launch dates starting after the minimum stay.
Wait on Mars until the return window.
Fly back to Earth, then plan the next outbound trip the same way.
With the default rules, the first launch is 31 October 2026, which falls in the real 2026 Earth–Mars launch window. A full round trip takes around four to five years, because after arriving at Mars the taxi usually has to wait about a year for Earth to come back into position.
The map
Planet orbits are drawn for Mercury, Venus, Earth and Mars at the current date.
The dashed arc is the planned route. Once the taxi launches, the flown part turns solid.
Hollow dotted rings show where the departure planet will be at launch and where the destination planet will be on arrival. They show why launches aim at where the target planet will be, not where it is now.
While the taxi is parked, dotted lines and an arc show the current angle between the planets. The status line compares it with the textbook Hohmann launch angle: Mars 44° ahead of Earth for the trip out, and Earth 75° behind Mars for the trip home.
Recent trips remain as faint paths.
Drag to pan, scroll or pinch to zoom.
The porkchop plot
The chart below the map shows every trajectory the planner evaluated for the current or next trip:
The horizontal axis is the launch date, covering 800 days from the earliest allowed launch.
The vertical axis is the flight time, from 90 to 450 days.
Colour shows the total Δv needed, from dark blue (cheapest) through green and yellow to orange. Trips more than 6 km/s above the cheapest option, and trajectories that are impractical, are left blank.
Faded cells break your current rules: flights longer than the limit, or trips over the fuel budget in "Fly fastest" mode.
The ring marks the trip the taxi will fly, the dashed line is the flight time limit, and the vertical line is the current date moving toward launch.
Hover over any point to see its launch date, flight time, arrival date and total Δv. Click a point to fly that trajectory instead. You can only change a trip that hasn't launched yet, and only to a launch date that hasn't passed.
Controls
Pause / Play stops and resumes the clock.
Skip to launch or Skip to arrival jumps to just before the next event.
Start over resets to today with a fresh service record.
Speed runs from 1 to 200 days per real second.
Taxi rules
Use least fuel picks the trajectory with the lowest total Δv.
Fly fastest picks the shortest flight whose total Δv fits the fuel budget per trip (5.5–20 km/s). If nothing fits, the taxi flies the cheapest trip and the panel says so.
Longest allowed flight (120–450 days) rules out slower trajectories.
Shortest stay on Mars and Shortest stay on Earth (0–700 days) set how long the taxi must stay before its next launch.
Changes to the flight limit, budget or mode immediately re-plan a trip that hasn't launched yet. Stay lengths apply from the next arrival.
Panels
Next trip / In flight lists launch and arrival dates, flight time, both engine burns, total Δv, and the speed relative to each planet on departure and arrival.
Service record counts trips, round trips, days in flight and total Δv used.
Log records launches, arrivals, upcoming windows and your changes.
How it works
Units and planet positions
Heliocentric calculations use astronomical units, years and G·M(Sun) = 4π². Speeds are converted to km/s for display (1 AU/year = 4.74047 km/s).
Planet positions and velocities come from JPL's approximate mean Keplerian elements for the J2000 epoch, advanced to each date with their rates of change per century. Kepler's equation is solved with Newton's method. The model is 2D: all orbits lie in the ecliptic plane and inclinations are ignored. The Earth entry uses the Earth–Moon barycentre.
Lambert's problem
Given two positions and the time to travel between them, Lambert's problem asks what orbit connects them. The solver uses the universal-variable formulation (Bate, Mueller & White; Curtis) with Stumpff functions, and finds the variable z by bisection, which is robust because flight time increases steadily with z for single-revolution transfers. It returns the spacecraft's velocity at departure and arrival. Only prograde, single-revolution transfers are considered, and trajectories that would escape the Sun are discarded.
Fuel cost
The speed relative to each planet on departure and arrival (v∞, the hyperbolic excess speed) is found by subtracting the planet's own velocity. Each burn is computed from a circular parking orbit using the Oberth-effect formula:
```
Δv = sqrt(v∞² + 2μ/r) − sqrt(μ/r)
```
The parking orbits are 400 km above Earth and 300 km above Mars. The total Δv for a trip is the departure burn plus the capture burn. Landing, launch from the surface and aerobraking are not included.
Planning a trip
For each trip the planner:
Evaluates a grid of launch dates every 4 days over 800 days, and flight times every 5 days from 90 to 450 days, about 14,600 Lambert solutions.
Picks the best grid point under the current rules.
Refines the choice with a finer search (half-day and one-day steps) around that point.
Planning takes a few hundred milliseconds or less. The chosen trajectory is converted to orbital elements and the taxi's position is computed from Kepler's equation at every frame, so its path is exact and lands precisely on the destination planet (tested to within 1e-13 AU).
Sample schedule
With the default rules (least fuel, flights up to 400 days, 30-day minimum stays), starting 17 September 2026:
Trip	Launch	Flight	Arrival	Total Δv
Earth → Mars	31 Oct 2026	311 days	7 Sep 2027	5.62 km/s
Mars → Earth	30 Aug 2028	316 days	12 Jul 2029	5.56 km/s
Earth → Mars	20 Jan 2031	257 days	4 Oct 2031	5.84 km/s
Mars → Earth	20 Jan 2033	244 days	21 Sep 2033	5.55 km/s
Switching to "Fly fastest" with a 9 km/s budget moves the first launch to 17 December 2026 and cuts the flight to 168 days, using 8.96 km/s.
Project structure
```
space-taxi.html          The entire application: markup, styles and script
space-taxi-README.md     This file
```
Customising
The script is split into an orbital core and the app:
`PLANETS` holds the orbital elements and colours.
`PARK` sets the parking orbit radius and gravitational parameter for each planet.
`G_DEP_STEP`, `G_DEP_SPAN`, `G_TOF_MIN`, `G_TOF_MAX` and `G_TOF_STEP` set the porkchop search grid.
`planLeg()` contains the choice logic; `better()` inside it defines what "best" means for each mode.
`STOPS` and `RANGE` set the porkchop colour scale.
Adding another destination, such as Venus, needs an entry in `PARK` and a change to which planets the taxi alternates between.
Limitations
Orbits are 2D, so plane-change costs are ignored. Real porkchop plots have a ridge of very expensive trajectories near 180° transfers that doesn't appear here.
Only single-revolution, prograde transfers are searched.
Planets' gravity only matters at the burns (patched conics); there are no gravity assists or mid-course corrections.
Planet positions use mean elements, accurate to within a few degrees roughly between 1800 and 2050.
Further reading
R. R. Bate, D. D. Mueller and J. E. White, Fundamentals of Astrodynamics, Dover, 1971.
H. D. Curtis, Orbital Mechanics for Engineering Students, Butterworth-Heinemann (Lambert's problem, chapter 5).
E. M. Standish, "Keplerian Elements for Approximate Positions of the Major Planets," JPL Solar System Dynamics.
NASA, "Mars Exploration: Launch Windows" and the Mars Design Reference Architecture 5.0 for real mission Δv figures.
