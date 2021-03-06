# Pencil Options
Pencil configuration files are written in YAML. When pencil starts up, it
searches for these files and loads them. See the main README.md for how
these files should look.

## General Configuration

These are options that go under the :config key in pencil configuration files.

* :graphite_url [String, required, no default]

  The url of your graphite instance.

* :url_opts [Hash, required, no default]

  A map of default graph options.

  In addition to <a href="#gopts">graph-level options</a>, an important default option
  you should set under :url_opts is

    :start: TIMESPEC

  TIMESPEC should be a
  [chronic](http://chronic.rubyforge.org/)-parsable time, and to be useful
  should be relative to the current time (e.g. "8 hours ago")

* :refresh_rate [Fixnum, optional, default 60]

  How often to refresh a changing view, in seconds.

  This doesn't apply to timeslices that aren't varying (i.e not current, see
  <a href="#threshold">:now_threshold</a>).

  Set this to false to disable automatic refreshing.

* :host_sort [["builtin", "numeric", "sensible"], optional, default "sensible"]

  Set to "builtin" to sort using ruby's builtin String sort.

  Set to "numeric" to sort hosts numerically (i.e. match secondarily on the
  first \d+).

  Set to "sensible" if you want to sort like this:

  http://www.bofh.org.uk/2007/12/16/comprehensible-sorting-in-ruby

* :quantum [Fixnum, optional, no default value]

  Map requests to NUM second intervals. Pencil floors request times to the
  minute, and does some modular arithmetic to do this mapping. This is
  especially useful for implementing a caching layer, so that many requests
  coming in near-simultaneously won't require graphite to generate different
  images for each request.

  Adding &noq=1 to a pencil url will disable this in case you need
  super-granularity for some reason, but you didn't hear it from me.

* :date_format [String, optional, default "%X %x"]

  strftime format for displaying dates.

* :metric_format [String, optional, default "%m.%c.%h"]

  The format your graphite metrics are stored in. For pencil to work your
  metrics need to be composed of three distinct pieces, concatenated in some
  regular fashion. The format strings are

  * %m metric
  * %c cluster
  * %h host

  If you want a literal %[mch] in your metric format string you likely have
  bigger problems than not being able to do so.

* <a name="threshold"/> :now\_threshold: [Fixnum, optional, default 300]

  How many seconds before Time.now an end time is considered to still be 'now',
  for the purposes of adding meta-refresh and displaying time intervals.

* :use_color: [Boolean, optional, default false]

  In graphite 0.9.8 and earlier coloring of targets sucks. Coloring is not done
  on a per-target basis, but rather by an additional parameter
  "colorList". Pencil handles colors by appending a target's color parameter
  (or a suitable default) onto this list. Targets are processed in order and
  consume the first color available in colorList.

  This sucks because nonexistent targets occasionally produced by pencil don't
  consume their associated color, which screws up the coloring of every
  subsequent metric.

  The latest Graphite trunk supports color as a property of a target, set with
  color(target, "COLOR"). If you are using a new graphite and want to take
  advantage of this more accurate coloring technique set :use_color to true,
  and bask in the glory of color correctness.

## <a name="gopts"/> Graph-level Options
This is a list of the supported graph-level options for pencil, which
correspond to request(image)-level options for graphite. These options are
key-value pairs, and are passed directly to graphite. Here is the list, with
minor annotations:

* vtitle: String (y-axis label)
* yMin: Fixnum
* yMax: Fixnum
* lineWidth: Fixnum (line thickness in pixels)
* areaMode: \[first, all, stacked\] (see graphite documentation)
* template: \[noc, alphas\] (alphas inverts colors)
* lineMode: staircase
* bgcolor: String
* graphOnly: bool (hide legend, axes, grid)
* hideAxes: bool
* hideGrid: bool
* hideLegend: bool
* fgcolor: String
* fontSize: Fixnum
* fontName: String (see your graphite instance for available fonts)
* fontItalic: bool
* fontBold: bool

## Target-level Options
This is a list of the supported target-level options for pencil. These are
mostly a list of transformations graphite supports, including summation and
scaling of metrics. You can apply them to individual metrics, or lists of
metrics. See the example configs for how this works. Also see the graphite
composer for the effects of these options, many of which are untested.

A note on the divideSeries case: for aggregate graphs, pencil takes the ratio
of the sums, as opposed to the sums of the ratios, for ease of implementation
and because it makes some sense. If you're not a fan of 80k URLs then you will
agree. divideSeries should only ever be used with two targets, like this:

    ? - stats.timers.mysql.innodb.threads_active_read.mean:
      - stats.timers.mysql.innodb.threads_total_read.mean:
    :
      !omap
      - :divideSeries:

(Notice the omap, as opposed to a hash, to impose an order in which options are
applied)

### Combinations
These functions take an arbitrary number of targets (usually simple metrics)
for arguments.

* sumSeries
* averageSeries
* minSeries
* maxSeries
* group

### Transformations
Some of these options take a single argument.

* scale
* offset
* derivative
* integral
* nonNegativeDerivative
* log BASE
* timeShift
* summarize
* hitcount

### Calculations
These functions take an arbitrary number of targets (usually simple metrics)
for arguments.

* movingAverage
* stdev
* asPercent
* diffSeries
* divideSeries (NOTE: takes exactly 1 series as divisor)

### Filters
Most of these options take a single argument.

* highestCurrent
* lowestCurrent
* nPercentile
* currentAbove
* currentBelow
* highestAverage
* lowestAverage
* averageAbove
* averageBelow
* maximumAbove
* maximumBelow
* sortByMaxima
* minimalist
* limit
* exclude

### Special Operations
* alias
* key (alias for alias)
* cumulative
* drawAsInfinite
* lineWidth
* dashed
* keepLastValue
* substr
* threshold
* color

Note: key is interpreted differently from the other options, which are more
simply translated.

color's interpretation depends on whether :use_color is set.
