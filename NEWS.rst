acpilight 1.2: 2019-06-22
--------------------------

* Added a new ``-perceived`` flag to control brightness in logarithmic
  steps (thanks to Lars-Dominik Braun).
* New action ``-get-steps`` to get the brightness resolution.
* Fixed the built-in udev rules for the keyboard backlight.


acpilight 1.1: 2018-06-21
--------------------------

* New upstream URL: https://gitlab.com/wavexx/acpilight
* Require python3 by default.
* A new Makefile is available for easier installation into standard
  locations (thanks to Nick Breen).
* Suggest/provide backward-compatible udev rules.
* Add compatibility notes in the manual page.
