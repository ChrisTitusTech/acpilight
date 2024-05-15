#!/usr/bin/env python3
# xbacklight: control backlight and led brightness on linux using the sys
#             filesystem with a backward-compatibile user interface
# Copyright(c) 2016-2019 by wave++ "Yuri D'Elia" <wavexx@thregr.org>
# -*- coding: utf-8 -*-
from __future__ import print_function, division, generators

APP_DESC = "control backlight brightness"
SYS_PATH = ["/sys/class/backlight", "/sys/class/leds"]

import argparse
from collections import OrderedDict
import os, sys
import time
from math import log10


def error(msg):
    print(sys.argv[0] + ": " + msg)

def get_controllers():
    ctrls = OrderedDict()
    for path in SYS_PATH:
        if os.path.isdir(path):
            for name in os.listdir(path):
                ctrls[name] = os.path.join(path, name)
    return ctrls

def clamp(v, vmin, vmax):
    return max(min(v, vmax), vmin)


class RawController(object):
    def __init__(self, path):
        self._brightness_path = os.path.join(path, "brightness")
        self._max_brightness = int(open(os.path.join(path, "max_brightness")).read())

    @property
    def max_brightness(self):
        return self._max_brightness

    @property
    def brightness_steps(self):
        return self._max_brightness + 1

    @property
    def brightness(self):
        return int(open(self._brightness_path).read())

    @brightness.setter
    def brightness(self, b):
        open(self._brightness_path, "w").write(str(int(round(b))))


class PcController(RawController):
    def __init__(self, path):
        super().__init__(path)

    @property
    def max_brightness(self):
        return 100

    @property
    def brightness(self):
        return super().brightness*100/super().max_brightness

    @brightness.setter
    def brightness(self, pc):
        bprop = super(PcController, self.__class__).brightness
        bprop.fset(self, pc*super().max_brightness/100)


class LogController(RawController):
    """
    Perceived brightness controls, based on this idea:
    https://konradstrack.ninja/blog/changing-screen-brightness-in-accordance-with-human-perception/
    """

    @property
    def max_brightness(self):
        return 100

    @property
    def brightness(self):
        if super().brightness == 0:
            return 0
        return log10(super().brightness)/log10(super().max_brightness)*self.max_brightness

    @brightness.setter
    def brightness(self, pc):
        bprop = super(LogController, self.__class__).brightness
        bprop.fset(self, 10**(pc/self.max_brightness*log10(super().max_brightness)))


def sweep_brightness(ctrl, current, target, steps, delay):
    sleep = (delay / 1000.) / steps
    for s in range(1, steps):
        pc = current + (target - current) * s / steps
        ctrl.brightness = pc
        time.sleep(sleep)
    ctrl.brightness = target


def pc(arg):
    if len(arg) == 0 or arg[0] not in '=+-0123456789':
        return None
    if arg[0] not in '=+-':
        arg = '=' + arg
    try:
        float(arg[1:])
    except ValueError:
        return None
    return arg


def main():
    ap = argparse.ArgumentParser(description=APP_DESC, add_help=False)
    g = ap.add_mutually_exclusive_group(required=True)
    g.add_argument("-h", "-help", action="help", help="Show this help and exit")
    g.add_argument("-list", action="store_true", help="List controllers")
    g.add_argument("-getf", action="store_true", help="Get fractional brightness")
    g.add_argument("-get-steps", action="store_true", help="Get brightness steps")
    g.add_argument("-get", action="store_true", help="Get brightness")
    g.add_argument("-set", metavar="PERCENT", type=float, help="Set brightness")
    g.add_argument("-inc", metavar="PERCENT", type=float, help="Increase brightness")
    g.add_argument("-dec", metavar="PERCENT", type=float, help="Decrease brightness")
    g.add_argument("pc", metavar="PERCENT", type=pc, nargs='?',
                   help="[=+-]PERCENT to set, increase, decrease brightness")
    ap.add_argument("-perceived", action="store_true", help="Use perceived brightness controls")
    ap.add_argument("-ctrl", help="Set controller to use")
    ap.add_argument("-time", metavar="MILLISECS", type=int,
                    default=200, help="Fading period (in milliseconds, default: 200)")
    g = ap.add_mutually_exclusive_group()
    g.add_argument("-steps", type=int,
                    default=0, help="Fading steps (default: 0)")
    g.add_argument("-fps", type=int,
                    default=0, help="Fading frame rate (default: 0)")
    ap.add_argument("-display", help="Ignored")
    args = ap.parse_args()

    # list controllers
    ctrls = get_controllers()
    if args.list:
        for name in ctrls:
            print(name)
        return 0

    # set current operating controller
    if args.ctrl is None:
        ctrl_path = next(iter(ctrls.values()))
    else:
        if args.ctrl not in ctrls:
            error("unknown controller '{}'".format(args.ctrl))
            return 1
        ctrl_path = ctrls[args.ctrl]

    if args.perceived:
        ctrl = LogController(ctrl_path)
    else:
        ctrl = PcController(ctrl_path)

    # uniform set arguments
    if args.pc is not None:
        v = float(args.pc[1:])
        if args.pc[0] == '=':
            args.set = v
        elif args.pc[0] == '+':
            args.inc = v
        elif args.pc[0] == '-':
            args.dec = v
    if args.fps:
        args.steps = int((args.fps/1000) * args.time)

    # perform the requested action
    if args.getf:
        print(ctrl.brightness)
    elif args.get:
        print(int(round(ctrl.brightness)))
    elif args.get_steps:
        print(int(ctrl.brightness_steps))
    else:
        current = ctrl.brightness
        if args.set is not None:
            target = args.set
        elif args.inc is not None:
            target = current + args.inc
        elif args.dec is not None:
            target = current - args.dec
        target = clamp(target, 0, ctrl.max_brightness)
        if current == target:
            pass
        elif args.steps <= 1 or args.time < 1:
            ctrl.brightness = target
        else:
            sweep_brightness(ctrl, current, target, args.steps, args.time)

    return 0


if __name__ == "__main__":
    try:
        sys.exit(main())
    except IOError as e:
        error(str(e))
        sys.exit(1)
