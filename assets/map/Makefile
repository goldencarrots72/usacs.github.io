RAW_MAP_FILES = us-states-geo.json
INTERMEDIATE_FILES = \
	us-states-projected-geo.json \
	us-states-projected-topo.json \
	test.svg
FINAL_PRODUCT = us-states-quantized-topo.json

DEPS = node_modules
BIN = ./node_modules/.bin
GEO2SVG = $(BIN)/geo2svg
TOPOQUANTIZE = $(BIN)/topoquantize
GEO2TOPO = $(BIN)/geo2topo
GEOPROJECT = $(BIN)/geoproject

US_STATES_GEO_JSON_URL = https://raw.githubusercontent.com/jgoodall/us-maps/master/geojson/state.geo.json
MAP_WIDTH = 750
MAP_HEIGHT = 400
MAP_SCALE = 700

all: $(FINAL_PRODUCT)

# use this target for deployment
prod: $(FINAL_PRODUCT)
	rm -rf $(INTERMEDIATE_FILES) $(RAW_MAP_FILES) $(DEPS)

# for quickly seeing what a geojson looks like
test: us-states-projected-geo.json $(DEPS)
	$(GEO2SVG) -w 960 -h 960 < $< > test.svg

# to further reduce filesize. some accuracy is lost
# so this can be tweaked to our liking
$(FINAL_PRODUCT): us-states-projected-topo.json $(DEPS)
	$(TOPOQUANTIZE) 1e5 < $< > $@

# a topo.json is preferred because they take up much less space
# than their geojson counterpart.
# see: https://github.com/topojson/topojson
us-states-projected-topo.json: us-states-projected-geo.json $(DEPS)
	$(GEO2TOPO) states=$< > $@

# pre-projecting the points will save a lot of time at runtime
us-states-projected-geo.json: us-states-geo.json $(DEPS)
	$(GEOPROJECT) 'd3.geoAlbersUsa().translate([$(MAP_WIDTH) / 2, $(MAP_HEIGHT) / 2]).scale($(MAP_SCALE))' < $< > $@

# grab the raw geojson for the US
us-states-geo.json:
	wget -O $@ $(US_STATES_GEO_JSON_URL)

$(DEPS): yarn.lock package.json
	yarn install

clean:
	rm -rf $(INTERMEDIATE_FILES) $(FINAL_PRODUCT)

# These targets are from when I thought this would be a world map, but
# a US map seems more appropriate. In the event we change our minds again,
# we'll have it.
COUNTRIES_TOPO_JSON_URL = https://gist.githubusercontent.com/abenrob/787723ca91772591b47e/raw/8a7f176072d508218e120773943b595c998991be/

countries-and-us-states-quantized-topo.json: countries-and-us-states-projected-topo.json
	topoquantize 1e5 < $< > $@

countries-and-us-states-projected-topo.json: countries-and-us-states-projected-geo.json
	geo2topo countries=$< > $@

countries-and-us-states-projected-geo.json: countries-and-us-states-geo.json
	geoproject 'd3.geoMercator().translate([($(MAP_WIDTH) / 2) + $(MAP_HORIZONTAL_OFFSET), ($(MAP_HEIGHT) / 2) + $(MAP_VERTICAL_OFFSET)])' < $< > $@

countries-and-us-states-geo.json: us-states-geo.json countries-geo.json
	geojson-merge $^ > $@

countries-geo.json: countries-topo.json
	topo2geo countries=$@ < $<

countries-topo.json:
	wget -O $@ $(COUNTRIES_TOPO_JSON_URL)

