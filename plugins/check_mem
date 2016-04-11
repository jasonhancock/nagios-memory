#!/usr/bin/perl

# Copyright (c) 2012 Jason Hancock <jsnbyh@gmail.com>
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is furnished
# to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
#
# This file is part of the nagios-cpu bundle that can be found
# at https://github.com/jasonhancock/nagios-memory

use strict;
use warnings;

use Nagios::Plugin;
use Nagios::Plugin::Threshold;

my $np = Nagios::Plugin->new(
    usage     => "Usage: %s [-w|--warning=<percent> ] [ -c|--critical=<percent> ]",
    shortname => 'MEMORY',
);

$np->add_arg(
    spec     => 'warning|w=s',
    help     => '-w, --warning=percent',
    required => 1,
);

$np->add_arg(
    spec     => 'critical|c=s',
    help     => '-c, --critical=percent',
    required => 1,
);

$np->getopts;

# read the data from /proc/meminfo into the %data hash
my %data;
open IN, '</proc/meminfo' or die('Can\'t read /proc/meminfo');
while(my $line=<IN>) {
    if($line=~m/^(.+):\s+(\d+)/) {
        $data{$1} = $2;
    }
}
close IN;

# Memory available for applications:
# the sum of the memory not used,
# the temporary storage for raw disk blocks (could be freed for programs by the kernel),
# in-memory cache for files read from the disk (the pagecache can also be tuned by the kernel if needed).
$data{'MemAvailable'} = $data{'MemFree'} + $data{'Cached'} + $data{'Buffers'};
# Memory used: the memory currently used by the applications, not considering the the raw buffers and the pagecache
$data{'MemUsed'} = $data{'MemTotal'} - $data{'MemAvailable'};

# Define thresholds
my %thresholds;
$thresholds{'Warning'} = $np->opts->warning  * .01 * $data{'MemTotal'};
$thresholds{'Critical'} = $np->opts->critical * .01 * $data{'MemTotal'};

$np->add_perfdata(
    label    => 'used',
    value    => $data{'MemUsed'} * 1024,
    uom      => 'B',
    warning  => $thresholds{'Warning'} * 1024,
    critical => $thresholds{'Critical'} * 1024,
    min      => 0,
    max      => $data{'MemTotal'} * 1024,
);

$np->add_perfdata(
    label    => 'cached',
    value    => $data{'Cached'} * 1024,
    uom       => 'B',
    warning  => undef,
    critical => undef,
    min      => 0,
    max      => $data{'MemTotal'} * 1024,
);

$np->add_perfdata(
    label    => 'buffers',
    value    => $data{'Buffers'} * 1024,
    uom      => 'B',
    warning  => undef,
    critical => undef,
    min      => 0,
    max      => $data{'MemTotal'} * 1024,
);

$np->add_perfdata(
    label    => 'free',
    value    => $data{'MemFree'} * 1024,
    uom      => 'B',
    warning  => undef,
    critical => undef,
    min      => 0,
    max      => $data{'MemTotal'} * 1024,
);

# check the result against the defined warning and critical thresholds,
# output the result and exit
my $code = $np->check_threshold(
    check    => $data{'MemUsed'} * 1024,
    warning  => $thresholds{'Warning'} * 1024,
    critical => $thresholds{'Critical'} * 1024,
);

$np->nagios_exit(
    return_code => $code,
    message     => sprintf(
	($np->opts->verbose ?
		'%d%% used (used: %d kB cached: %d kB buffers: %d kB free: %d kB)' :
		'%d%% used'),
        ($data{'MemUsed'} / $data{'MemTotal'}) * 100,
        $data{'MemUsed'},
        $data{'Cached'},
        $data{'Buffers'},
        $data{'MemFree'},
    )
);
