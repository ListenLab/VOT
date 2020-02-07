# VOT
Praat script to manipulate VOT in natural speech

The script walks the user through a series of steps whereby pre-existing sounds (e.g. “deer” and “tier”) are used to generate a continuum varying by VOT and other related properties. The user initiates a startup window where parameters are declared. Basic parameters include the range of VOT and the number of continuum steps. These are likely the most essential (and perhaps the only) parameters of interest to most readers of this paper. But there are other relevant factors such as starting F0 values for the voiced and voiceless continuum endpoints, whether F0 should change independently or in conjunction with VOT, and the duration (in milliseconds) over which any change in F0 will be imposed on the F0 contour in the syllable. In some cases, the default values of the script are motivated by generally realistic ranges of cues found in natural utterances. In other cases, the values are motivated by best guesses as to what will maintain natural sound quality; the experimenter is able to decide which of these parameters are deserving of extra scrutiny or control. In a case of exploration of a cue without extensive previous literature, the experimenter is encouraged to collect goodness or naturalness ratings along with identification labels. 

## Where to find the tutorial paper:
https://asa.scitation.org/doi/abs/10.1121/10.0000692

Note: the supplemental materials on this page that have more helpful filenames. 

---------------------------------------------------
Three basic strategies:
  1) Control VOT and F0 independently

  2) Control VOT, have a constant level of F0: 
       enter only 1 step of F0
       and type in that constant F0 level as the "min" F0

  3) Co-vary VOT and F0 by having a different F0 for each step of VOT
       (This sounds most natural, and is intended for when
        you aren't interested in specific cue weighting, 
         but are interested
        simply in the perception of voicing for demonstration)
     
 Inspect and modify some variables at the end of the script. 
    ... such as controlling proportional VOT vowel cutback, etc. 

 Options to shape VOT and vowel amplitude contours at segment boundary
    This is highly recommended. Otherwise, medium-lag sounds will sound goofy
     because there isn't a neat envelope transition from the burst
     to the vowel (this is especially noticable for /t/ sounds)

 You can prepare a pre-made aspiration sound
   - If you want to modify it in strange ways before making the continuum
   - ... or if you want to ensure that multiple different continua
    have the same exact VOT segments. 
   - ... or if you want to do funny things like put the wrong aspiration on a segment
       ... or mix bursts and aspirations.

'new_output_folder' names the new sub-folder that will be created in the parent directory
  (the parent directory should be a folder that **already exists** on your computer)

 'Sound_object_name_prefix' is a constant string prefix for each sound for output naming

 Note 1: This script is a mixture of different styles of Praat coding. 
 Note 2: Requesting negative VOT values will initate creation of a "prevoicing" segment,
	 	... which is not validated as a perfect match to natural prevoicing,
		... but which I think sounds pretty good. 

# Log of notable updates:
 Version 18: supports pre-voicing (preserves burst), warns against RMS equalization. 
 Version 19: New startup window with different organization
 Version 26: New startup window with different organization, 
		organized procedure files in order,
		check to make sure parent folder exists,
		and that the subfolders don't already exist
		changed some procedure names for transparency
		Check for adequate version of Praat
# Version 30: 
    object selection by index instead of name
		saving functions changed to probably work across platforms
		   without weird string combinations
		functions changed to accept object-index inputs 
		   rather than object names
		slight change in names for saved files
		Change in ordering of resampling procedures
		Saves original aspiration before lengthening adjustment
		Offers the option to clean up files
		   if you're creating lots of them
		Easier-to-read end-of-script options
		   for things like cutback ratio, forced extra control, etc. 
		Better documentation overall 
# Sample sounds
DT_Deer.wav and DT_Tier.wav are sample sounds that you can use to test out the script. The current version of the script will automatically select teh correct landmarks for these sounds. 

# Is this the most current version?
New versions starting with v30 will be uploaded when available. Whichever is the most current will be named with "_current_version.txt_" in the filename. 
