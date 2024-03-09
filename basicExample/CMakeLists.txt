file(GLOB ofxSampleHeaders *.h)
file(GLOB ofxSampleSources *.cpp)
target_sources(ofxSample PUBLIC ${ofxSampleHeaders} ${ofxSampleSources})
