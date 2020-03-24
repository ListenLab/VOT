###########################################################################
###########################################################################
## Make VOT continuum
##
## Matthew B. Winn 
## University of Minnesota
##
## Version 31
## March 23, 2020
##
## Three basic strategies:
##   1) Control VOT and F0 independently
##
##   2) Control VOT, have a constant level of F0: 
##        enter only 1 step of F0
##        and type in that constant F0 level as the "min" F0
##
##   3) Co-vary VOT and F0 by having a different F0 for each step of VOT
##        (This sounds most natural, and is intended for when
##         you aren't interested in specific cue weighting, 
##          but are interested
##         simply in the perception of voicing for demonstration)
##      
##  Inspect and modify some variables at the end of the script. 
##     ... such as controlling proportional VOT vowel cutback, etc. 
##
##  Options to shape VOT and vowel amplitude contours at segment boundary
##     This is highly recommended. Otherwise, medium-lag sounds will sound goofy
##      because there isn't a neat envelope transition from the burst
##      to the vowel (this is especially noticable for /t/ sounds)
## 
##  You can prepare a pre-made aspiration sound
##    - If you want to modify it in strange ways before making the continuum
##    - ... or if you want to ensure that multiple different continua
##     have the same exact VOT segments. 
##    - ... or if you want to do funny things like put the wrong aspiration on a segment
#       ... or mix bursts and aspirations.
##
## 'new_output_folder' names the new sub-folder that will be created in the parent directory
##   (the parent directory should be a folder that **already exists** on your computer)
##
##  'Sound_object_name_prefix' is a constant string prefix for each sound for output naming
##
##  Note 1: This script is a mixture of different styles of Praat coding. 
##  Note 2: Requesting negative VOT values will initate creation of a "prevoicing" segment,
##	 	... which is not validated as a perfect match to natural prevoicing,
##		... but which I think sounds pretty good. 
##
##  Log of notable updates:
##  Version 18: supports pre-voicing (preserves burst), warns against RMS equalization. 
##  Version 19: New startup window with different organization
##  Version 26: New startup window with different organization, 
##		organized procedure files in order,
## 		check to make sure parent folder exists,
##		and that the subfolders don't already exist
##		changed some procedure names for transparency
##		Check for adequate version of Praat
##  Version 30: object selection by index instead of name
##		saving functions changed to probably work across platforms
##		   without weird string combinations
##		functions changed to accept object-index inputs 
##		   rather than object names
##		slight change in names for saved files
##		Change in ordering of resampling procedures
##		Saves original aspiration before lengthening adjustment
##		Offers the option to clean up files
##		   if you're creating lots of them
##		Easier-to-read end-of-script options
##		   for things like cutback ratio, forced extra control, etc. 
##		Better documentation overall 
## Version 31: Correct a variable name that would cause an error 
###########################################################################
###########################################################################


form Enter settings for VOT / F0 continuum 

	comment Enter VOT settings - number of steps, high (voiceless) & low (voiced) VOT endpoint values
		natural number_of_VOT_steps 7
		real left_vot_min_and_max 10
		real right_vot_min_and_max 70

	optionmenu Method_of_choosing_aspiration: 1
            option Select VOT aspiration from original words
            option Use a pre-made VOT aspiration segment

	comment ------------------------------------------------------------------------------
	comment Enter number of F0 steps, and the max & min F0 values (if only one step, enter same value)
		natural f0_steps 5
		real left_min_and_max_F0_onset -5
		real right_min_and_max_F0_onset 40
        
	optionmenu F0_manipulation: 1
		option relative to original F0
		option absolute Hz

	optionmenu VOT_and_F0_covariance: 2
	    	option Control VOT and F0 separately
            	option Covary F0 and VOT in tandem
        	
		real f0_time_range_in_ms 75

	comment ------------------------------------------------------------------------------
		boolean covary_aspiration_intensity 1
	
		real aspiration_intensity_range 6
	comment ------------------------------------------------------------------------------

	optionmenu save_files: 2
		option do not save any files
		option save all the files and info
		
	comment Enter directory path for the new files (a folder that already exists) (if you're saving output)
		sentence parent_folder C:/Users/Matt/Desktop/Speech_continua

		sentence new_output_folder VOT_DT
	comment ------------------------------------------------------------------------------
		sentence Sound_object_name_prefix DT

endform

#
#
##
###
#####
########
#############
#####################
##################################
####################################################### 
## THE SEQUENCE OF ACTIONS #####################################
####################################################### 
##################################
#####################
#############
########
#####
###
##
#
#

#======================================#
# Preparation
clearinfo
call check_praat_version
call set_variables_at_startup

#======================================#
# User-controlled actions
	# Select the starting objects
	# Next line hard-coded for DT demo;
	# it won't cause an error if there's no DT_Deer sound. 
		nocheck select Sound DT_Deer
		pause Select the voiced-onset sound
		voiced$ = selected$ ("Sound")
		voiced_sound_index = selected("Sound")

		nocheck select Sound DT_Tier
		pause Select the voiceless-onset sound
		voiceless$ = selected$ ("Sound")
		voiceless_sound_index = selected("Sound")
		
	# Print to the info window
		print ###############################################'newline$'
		print SOUND SELECTION INFO'newline$'
		print User selected 'voiced$' as the voiced endpoint'newline$'
		print User selected 'voiceless$' as the voiceless endpoint'newline$'
		print 'newline$'###############################################'newline$'
	

	# Match sampling frequency to the highest of the two sounds
		call match_samplerates_by_index voiced_sound_index voiceless_sound_index
		# restore indexing variables
			voiced_sound_index     = match_samplerates_by_index.sound1
			voiceless_sound_index  = match_samplerates_by_index.sound2


	# Select the aspiration segment that will be manipulated 
	#  into a contiuum of aspiration durations
		call select_aspiration_segment

	# Mark the onset of the vowel in the voiced-onset sound
		call mark_vowel_onset_ind 'voiced_sound_index'

#======================================#
# Automatic actions 

# Set levels of VOT, F0, intensity, etc. 
        call generate_all_continuum_values

# Make the aspiration portions
        call make_VOT_aspiration_sounds
        
# Make the vowel portions
    	call make_silent_buffer
        call create_F0_manipulation_object
        call make_VOT_cutbacks
        
# Put the pieces together and construct the continuum
        call modify_and_assemble_components_and_save

# Save info files (does nothing if you didn't check the save box)
        call save_info_files

# cleanup remaining files
	call cleanup_remaining

# Assemble the continuum to view, if it's a small series
	call assemble_continuum_series_to_view

pause Done!
## Finished :)	

#
#
##
###
#####
########
#############
#####################
##################################
####################################################### 
## P R O C E D U R E S ##################################################
#######################################################
##################################
#####################
#############
########
#####
###
##
#
#

procedure check_praat_version
	#>> Ideally it's the most recent version...
	#>> But I think that at least version  5.3.64 will do. 
	praat_version$ = left$(praatVersion$, (rindex(praatVersion$, ".")-1));
	praat_version = 'praat_version$'
	
	print Using Praat version 'praat_version$''newline$'
	
	if praat_version < 6
	   exit Please download the most recent version of Praat.
	endif
endproc


procedure check_if_parent_folder_exists parent_folder$
	#>> If the user wants to save the files, 
	#>> Check if the user declared a valid parent folder to save them.
	#>> It will do nothing if the directory doesn't exist
	#>> (i.e. it won't write to a mysterious temp directory; it will throw an error,
	#>> but the error won't be visible here because of the 'nocheck')

	if save_files > 1
		temp_filename$ = parent_folder$ + "/" + "my_temporary_Praat_file.txt"

		# write a temporary txt file to the parent folder
		nocheck writeFileLine: temp_filename$, "This is just to check if the directory exists"

		# can the file be found?
		file_exists = fileReadable (temp_filename$)

		if file_exists  == 1
			# if you *could* read that file,
			# then you can delete it,
			# and this is confirmation that the directory is valid. 
			deleteFile: temp_filename$
			print Saving to parent folder 'parent_folder$''newline$'

		else
			# if that file wasn't readable, that means that the directory wasn't valid. 
			print 'newline$'No filepath found
			exit Your parent directory doesn't exist. Check spelling. The parent directory must *already* exist. 
		endif
	endif
endproc


procedure check_if_subfolders_already_exist
	#>>  check if this sub-directory has *already* been made
	#>> and give the user a chance to stop the script
	#>> before it over-writes previously created files
	
	# first see if there are already files in the "new" sub-folder
	Create Strings as directory list: "directoryList", parent_folder$ + "/" + new_output_folder$
	num_subfolders = Get number of strings
	
	# If that folder already contains stuff, 
	# then stop and ask the user if they want to proceed
	if num_subfolders > 0 
		sub_folder_names$ = ""
		for index from 1 to num_subfolders
			sub_folder_name$ = Get string: index
			if index > 1
				sub_folder_name$ = ", " + "'sub_folder_name$'"
			endif
			# add to the list of current sub-folder names
			sub_folder_names$ = "'sub_folder_names$'" + "'sub_folder_name$'"
		endfor
		
		beginPause: "'parent_folder$' already includes sub-folder 'new_output_folder$' with 'sub_folder_names$'"
		comment: "Proceeding might over-write files that already exist!"
		comment: "How do you want to proceed?"
		clicked = endPause: "Continue anyway (risky)", "Exit now", 0
		
				
		if clicked == 2
			# if user wants to exit, exit right away
			select Strings directoryList
			Remove
			exit
		endif
	endif
	
	# remove the object used to check if the sub-folders existed
	select Strings directoryList
	Remove
endproc


procedure make_save_directories
	#>> Makes new directories 
	#>> one as the main directory 
	#>> one for the stimuli, and one for the original components
	
	# Different method recommended by Paul Boersma
	createDirectory (parent_folder$ + "/" + new_output_folder$)
	createDirectory (parent_folder$ + "/" + new_output_folder$ + "/Stimuli")
	createDirectory (parent_folder$ + "/" + new_output_folder$ + "/Original_sounds")

	# Alert the user
	new_folder_to_display$ = parent_folder$ + "/" + new_output_folder$
	pause Directories made in 'new_folder_to_display$'
endproc


procedure select_aspiration_segment
	#>> Asks the user to select the aspiration segment
	#>> that will be used to create the continuum. 
	#>> Assigns the variable string name 'full_aspiration_sound$'
	#>> to the full-duration segment
	#>> either by selecting from an original sound,
	#>> or by choosing an object in the Objects list

	if preselected = 1
		# You already have a prepared aspiration segment
			pause Click on the sound to be used for the VOT burst & aspiration
			# Whatever the name of the object is, 
			# that's now the value that shows up when you query 'full_aspiration_sound$'
			full_aspiration_sound$ = selected$ ("Sound")
			full_aspiration_sound = selected("Sound")
			original_aspiration_sound = selected("Sound")
			print used 'full_aspiration_sound$' for the aspiration noise'newline$'

	elsif preselected = 0
		# Select a segment from the voiceless-onset sound
			call select_aspiration_from_original_ind 'voiceless_sound_index' vot_segment
			# establish variables with the newly-created sound object
			# from that procedure
			full_aspiration_sound$ = selected$("Sound")
			full_aspiration_sound = selected("Sound")
			original_aspiration_sound = selected("Sound")

	endif

	# Get some info to be used later
	raw_aspiration_duration = Get total duration
	aspiration_intensity_orig = Get intensity (dB)

	# If the length of the aspiration segment 
	# doesn't support the entire VOT continuum,
	# it is lengthened and re-cross-faded; 
	# extensive lengthening (2.4x) forbidden. 
		# first initiate this variable for later
		aspiration_modified = 0

	# calculate how much you'd need to modify the aspiration
		requiredDuration = vot_max + vot_crossfade_duration
		if raw_aspiration_duration < requiredDuration
			call lengthen_special_ind 'full_aspiration_sound' requiredDuration lengthened_VOT
			
			# save the original aspiration for posterity
			select full_aspiration_sound
			if save_files > 1
				Save as WAV file: "'parent_folder$'/'new_output_folder$'/Original_sounds/" + "original_aspiration_sound_before_lengthening.wav"
			endif

			# Re-establish variable name for the full aspiration
			select Sound lengthened_VOT
			full_aspiration_sound$ = selected$("Sound")
			full_aspiration_sound = selected("Sound")
		endif
endproc


procedure select_aspiration_from_original_ind .sound .newname$
	#>> selecting the aspiration segment from among the original sounds 
	#>> you should have declared the first argument to be 'voiceless$'

	select .sound
	.sound$ = selected$("Sound")
	View & Edit

	editor Sound '.sound$'
	# Mark aspiration ONSET
	# Hard-coded for DT Deer-Tier example
	Select... 0.05 0.05
		pause Click on the START of the burst in the voiceless consonant
		Move cursor to nearest zero crossing
		aspiration_start = Get cursor

	# Mark aspiration OFFSET
	# Hard-coded for DT Deer-Tier example
	Select... 0.121 0.121
		pause Click on the END of the aspiration in the voiceless consonant
		Move cursor to nearest zero crossing
		aspiration_end = Get cursor
		Close
		endeditor

	# Extract the aspiration 
	select .sound
	Extract part: aspiration_start, aspiration_end, "rectangular", 1, "no"	
	Rename... '.newname$'

	print ###############################################'newline$'
	print VOT CONTINUUM INFO'newline$'
	print 'newline$'###############################################'newline$'
	print User selections'newline$'
	print Selected an aspiration segment from 'voiceless$' sound'newline$'
	print    with onset landmark at 'aspiration_start''newline$'
	print    and offset landmark at 'aspiration_end''newline$'
endproc


procedure lengthen_special_ind .sound .required_duration .newname$
	#>> "Lengthen" by simply taking the last third
	#>> and appending it to the end of the original segment
	#>> this is ugly, but sounds better in the case of working with aspiration segments,
	#>> because the 'Lengthen' procedure produces spurious amplitude modulations. 

	# first check to see if it's feasible
	select .sound
	.name$ = selected$("Sound")
	.orig_duration = Get total duration
	.extra_needed = .required_duration - .orig_duration
	.multiplier = .required_duration/.orig_duration
	original_aspiration_sound = selected("Sound")
	
	print 'newline$'###############################################'newline$'
	
	if .multiplier > 2.4
		# Not feasible to lengthen by a factor of more than 2.4;
		# it would sound really choopy and unnatural
		print required duration is '.required_duration:3' and your total sound duration is '.orig_duration:3' :( 'newline$'
		print lengthening your segment by the appropriate amount ('.multiplier:3') will cause too much distortion. 
		exit Sorry, the chosen sound is not long enough to support the VOT continuum you want :(      See Info Window for details. 

	elsif .extra_needed < (.orig_duration * 0.4)
		# If you need only a little longer aspiration than you have,
		# Simply append the last 0.4 (or less) to the original sound)
		# instead of running the lengthen procedure
		print Replicating the last '.extra_needed:3' at the end of the aspiration segment'newline$'
		print    to support the maximum length of the VOT continuum'newline$''newline$'
			select .sound
			# extract 30ms more than needed...
			Extract part: (.orig_duration-.extra_needed)-0.03, .orig_duration, "rectangular", 1, "no"
			Rename... temp_offset

			# ... Blend it in smoothly with 30ms overlap
			select .sound
			plus Sound temp_offset
			Concatenate with overlap... 0.03
			Rename... junk
			Extract part: 0, .required_duration, "rectangular", 1, "no"
			Rename... '.newname$'
			.new_sound = selected("Sound")
			aspiration_modified = 1

			select Sound junk
			plus Sound temp_offset
			Remove
	elsif .extra_needed > (.orig_duration*0.4)
		# if you need a substantially longer aspiration,
		# run the lengthning procedure
			print using regular Lengthen procedure 'newline$'
			print using multiplier of '.multiplier:3''newline$'
			select .sound
			Lengthen (overlap-add)... 100 300 .multiplier
			Rename... '.newname$'
			.new_sound = selected("Sound")
			aspiration_modified = 1
	else
		print No lengthening required for the aspiration segment to support maximum VOT in the continuum'newline$'
	endif

	select .new_sound
	# VOT segment has been lengthened to accomodate your continuum	   
endproc


procedure mark_vowel_onset_ind .sound
	#>> Asks the user to click on the vowel onset
	#>> i.e. where the periodicity begins
		select .sound
		.sound$ = selected$("Sound")
		View & Edit
		editor Sound '.sound$'
			# Hard-coded for DT Deer-Tier example
			Select... (0.05416757235438554) (0.05416757235438554)
			pause click on the vowel onset
			Move cursor to nearest zero crossing
			vowel_start_point = Get cursor
			Close
		endeditor
	print 'newline$'###############################################'newline$'
	print Marked 'vowel_start_point' as the vowel onset in the '.sound$' sound'newline$'
endproc


procedure match_samplerates_by_index .sound1 .sound2
	#>> If two sounds have different sampling rates,
	#>> up-sample the lower-sampled one. 
	
		select .sound1
		.sound1$ = selected$("Sound")
		.samplerate1 = Get sampling frequency
		numchannels = Get number of channels
		
		select .sound2
		.sound2$ = selected$("Sound")
		.samplerate2 = Get sampling frequency
		
		if .samplerate1 > .samplerate2
			samplerate = .samplerate1
			select .sound2
			resampled_sound2 = Resample... '.samplerate1' 50
			# line to update info window
			print 'newline$'Upsampled '.sound2$' to '.samplerate1' Hz to match '.sound1$''newline$'
			
			select .sound2
			Remove
			select resampled_sound2
			Rename... '.sound2$'
			# restore indexing variable
			.sound2 = selected("Sound")
		
		elsif .samplerate2 > .samplerate1
			samplerate = .samplerate2
			select .sound1
			resampled_sound1 = Resample... '.samplerate2' 50
			# line to update info window
			print 'newline$'Upsampled  '.sound2$' to '.samplerate1' Hz to match '.sound1$''newline$'
			
			select .sound1
			Remove
			select resampled_sound1
			Rename... '.sound1$'
			# restore indexing variable
			.sound2 = selected("Sound")
		else
			samplerate = .samplerate1
		endif
endproc


procedure generate_all_continuum_values
	#>> Declare all the continuum values for VOT, F0, and aspiration intensity
	#>> for use in the script, and for printing to the info window
	# Print VOT continuum values
	
		
		print 'newline$'
		print 'newline$'###############################################'newline$'
		print VOT steps 'newline$'
		call generate_continuum_levels number_of_VOT_steps vot_min vot_max VOT_ 1 3
		print 'newline$'
		# yields variable levels with the name step_VOT_'n'
		
		max_cutback_used = step_VOT_'number_of_VOT_steps' * vowel_cutback_to_VOT_ratio		
		
		# info for printing to the window:
			extra_aspiration_added = vot_crossfade_duration/2
			extra_aspiration_added = extra_aspiration_added * 1000
			extra_aspiration_added = 'extra_aspiration_added:0'
		
		print Added 'extra_VOT_silence_buffer' silence at the onset of each sound'newline$'
		print Added 'extra_aspiration_added' ms aspiration at the end for blending into the vowel'newline$'
		print 'newline$'
		print Vowel-cutback information:'newline$'
		print For each 1 ms of VOT, 'vowel_cutback_to_VOT_ratio' ms of vowel cutback'newline$'
		print ('vowel_cutback_to_VOT_ratio' vowel-cutback ratio)'newline$'
		print Maximum vowel cutback actually used: 'max_cutback_used:3''newline$'
		print Maximum vowel cutback duration permitted: 'max_cutback_dur' seconds'newline$'

	print 'newline$'
	print 'newline$'###############################################'newline$'
        # If you're doing relative F0,
        # obtain first F0 point here, 
        # so that you can send the relative values 
        # into the generate_continuum values procedure
        if f0_manipulation_style$ == "relative_change_from_original"
            
		call check_F0_tracking
		# yields variable 'f0_onset'
		
		# Update the min and max values relative to this measurement
		min_F0_onset = min_F0_onset + f0_onset
		max_F0_onset = max_F0_onset + f0_onset

		print Original sound F0 onset: 'f0_onset:1''newline$'
		print F0 continuum minimum:  'min_F0_onset:1''newline$'
		print F0 continuum maximum:  'max_F0_onset:1''newline$'
		print Modification declared as relative change from the original'newline$'

	elsif f0_manipulation_style$ == "absolute_Hz"
		min_F0_onset = min_F0_onset
		max_F0_onset = max_F0_onset

		print Original sound F0 onset: 'f0_onset:1''newline$'
		print F0 continuum minimum:  'min_F0_onset:1''newline$'
		print F0 continuum maximum:  'max_F0_onset:1''newline$'
		print Modification declared as explict declaration of F0 in 'f0_manipulation_style$''newline$'
        endif
        
	# Print F0 continuum values
		print 'newline$'###############################################'newline$'
		print F0 steps: 'newline$'
		if number_of_f0_steps > 1
			call generate_continuum_levels number_of_f0_steps min_F0_onset max_F0_onset F0_ 1 0
			print 'newline$'
			# yields variable levels with the name step_F0_'n'
		else
			step_F0_1 = min_F0_onset
			call generate_continuum_levels number_of_f0_steps min_F0_onset min_F0_onset F0_ 1 0
			print F0 onset:'tab$''min_F0_onset:0' 'newline$'
		endif
		# Pitch settings
			print Pitch analysis between 'minpitch' and 'maxpitch' Hz'newline$'

	# Print aspiration intensities
		print 'newline$'###############################################'newline$'
		print Aspiration intensities (dB)'newline$'
		call generate_continuum_levels number_of_VOT_steps aspiration_intensity_orig-aspiration_intensity_range aspiration_intensity_orig aspiration_int_ 1 1

		print 'newline$'
		print Aspiration change (dB)'newline$'
		call generate_continuum_levels number_of_VOT_steps -aspiration_intensity_range 0 aspiration_int_change_ 1 2
		print 'newline$'
        
        # Crossfade setting
        	print 'newline$'###############################################'newline$'
        	print Used 'vot_crossfade_duration' as the VOT cross-fade duration 'newline$'
        		
        # Intensity setting
        	print 'newline$'###############################################'newline$'
		if scale_to_equal_RMS$ == "no"
			print Output sounds maintained natural intensity-VOT relationship'newline$'
			print (not equalized for RMS)'newline$'
		else
			print output sounds equalized for RMS intensity; not equalized for loudness'newline$'
			output intensity: 'equal_RMS_intensity_target''newline$'
		endif
        	        	
        	if shape_vowel_envelope_onset == 1
        		print 'newline$'###############################################'newline$'
        		print Vowel onset intensitying ramping'newline$'
endproc


procedure generate_continuum_levels .steps .low .high .prefix$ .printvalues .precision
	#>> Takes continuum endpoints and number of steps,
	#>> and calculates the interpolated points.
	#>> It also assigns an array of variable names 
	#>> (with user-set variable name prefix),
	#>> Prints them to the info window with set decimal precision

	# Loop through continuum indices
	for thisStep from 1 to .steps
		# This step has to be split into two steps
		#
		# 1. First calculate the interpolated value
		temp = (('thisStep'-1)*('.high'-'.low')/('.steps'-1))+'.low'

		# 2. Dynamically declare a variable name
		# and assign the temp value
		# e.g. step_VOT_3, step_VOT_4, step_VOT_5, etc.
		step_'.prefix$''thisStep' = temp
		.value = step_'.prefix$''thisStep'

		if .printvalues = 1
			# Prints values with n digits of precision
			.value_print = round( .value * 10^.precision )/( 10^.precision )
			print '.prefix$''thisStep''tab$''.value_print' 'newline$'
		endif
	endfor
endproc


procedure check_F0_tracking
	#>> Ensure that F0 tracking works,
	#>> because if you try to manipulate F0,
	#>> the manipulation will only work for regions 
	#>> that were properly tracked.
	
	# Where to extract the first F0 reading
        	f0_start_point = vowel_start_point + (vot_crossfade_duration/2)
        
        # Set an original timing landmark
		.temp_landmark = f0_start_point
		select Sound 'voiced$'
		To Pitch... 0 minpitch maxpitch
		f0_onset = Get value at time... f0_start_point Hertz Linear

		# If the F0 at that point is undefined,
        # then scroll along the sound until you find a defined point
		while f0_onset = undefined
			f0_start_point = f0_start_point + 0.001
			f0_onset = Get value at time... f0_start_point Hertz Linear
		endwhile

	# Keep track of the amount of time advancement needed to find a pitch value
		time_elapsed_until_f0_detection = f0_start_point - .temp_landmark

	# If the pitch can't be tracked within 25 ms of the user-selected landmark,
	# the user must define the pitch pulse tracking 
	# in the manipulation object
		if time_elapsed_until_f0_detection > 0.025
			Edit
			editor Manipulation 'voiced$'
				beginPause ("Ensure all blue pulses are marked.")
				comment ("(ctrl+p to add pulses where there are missing blue lines)")
				comment ("")
				comment ("If you don't know what to do,")
				comment ("then consider working with a different sound recording")
				comment ("that has better pitch tracking.")
				endPause ("Okay, I'm done",1)
				Close
			endeditor
		endif
		select Pitch 'voiced$'
		Remove
endproc


procedure make_VOT_aspiration_sounds
	 #>> Pulls out the appropriate duration of aspiration noise
	 #>> from the full-length aspiration
	
	# First, make a little bit of silence to append and preappend to the cutback vowel
    		call make_silence extra_VOT_silence_buffer extra_VOT_silence_buffer 1

	# Loop through continuum indices
	for thisStep from 1 to number_of_VOT_steps
		vot_duration_this_step = step_VOT_'thisStep'
		if vot_duration_this_step > 0
			# POSITIVE VOT
			# If the VOT is at least zero, 
			# Takes the full aspiration segment and chops it
			# to the length specified for this continuum step

			# Sets VOT duration, plus extra blend time for cross-fading
				vot_duration_this_step = step_VOT_'thisStep' + (vot_crossfade_duration/2)
				call extract_VOT_segment_ind 'full_aspiration_sound' 'thisStep' 'vot_duration_this_step'

			# Intensity adjustment (optional)
			if covary_aspiration_intensity = 1
				select Sound VOT_aspiration_'thisStep'
				# dB adjustment is step_aspiration_int_change_'thisStep'
				Multiply... 10^(step_aspiration_int_change_'thisStep' / 20)
			endif
		else
			# PREVOICING
			# If it's a negative VOT, make prevoicing 
			# first make the murmur
				call make_prevoicing_ind -vot_duration_this_step 'voiced_sound_index' 'vowel_start_point' 'thisStep' prevoicing_extracted

			# Prepare silent interval to introduce prevoicing
			# Leave 3ms for them to overlap. 
				call make_silence buffer_to_preappend -(vot_duration_this_step-0.001) numchannels

			# Extract just the burst (the first 5ms)
				call extract_VOT_segment_ind 'full_aspiration_sound' 'thisStep' (0.005+(vot_crossfade_duration/2))
				select Sound VOT_aspiration_'thisStep'
				Rename... temp_burst_w_onset_buffer
				temp_duration = Get total duration

			# Remove the absolute buffer
				Extract part... extra_VOT_silence_buffer temp_duration rectangular 1 no
				Rename... temp_burst

			# Prepare another buffer to re-introduce later
				call make_silence buffer_to_restore extra_VOT_silence_buffer numchannels

			# Burst intensity adjustment
			# ... if you chose that option
				if covary_aspiration_intensity == 1
					if covary_even_for_prevoicing == 1
						select Sound temp_burst
						# dB adjustment is step_aspiration_int_change_'thisStep'
						Multiply... 10^(step_aspiration_int_change_'thisStep' / 20)        
					endif
				endif

			# Add silence to the onset of the burst 
			# for when it's added to the prevoicing
				select Sound buffer_to_preappend
				plus Sound temp_burst
				Concatenate
				Formula... self[col] + Sound_prevoicing_extracted[col]
				Rename... temp_burst_w_silent_buffer_for_prevoicing

			# Re-introduce absolute onset silence buffer
				select Sound buffer_to_restore
				plus Sound temp_burst_w_silent_buffer_for_prevoicing
				Concatenate

			# Rename according to same style as for positive VOT
				Rename... VOT_aspiration_'thisStep'

			# Cleanup
				select Sound prevoicing_extracted
				plus Sound temp_burst_w_onset_buffer
				plus Sound temp_burst
				plus Sound buffer_to_preappend
				plus Sound buffer_to_restore
				plus Sound temp_burst_w_silent_buffer_for_prevoicing
				Remove
	    endif
    	endfor

	if extracontrol == 1
		# User can inspect the aspiration continuum
		pause Check vot continuum steps to make sure they all seem reasonable
	endif
endproc


procedure make_silence .name$ .duration .numchannels
	#>> Wrapper for make sound function
	Create Sound from formula... '.name$' '.numchannels' 0 '.duration' 'samplerate' 0
endproc


procedure extract_VOT_segment_ind .sound .number .newduration
	#>> Pulls out a segment from a longer sound
	#>> and adds the silent buffer at the onset
    	#>> yields Sound obejct named VOT_'.number'
    	
	select .sound
	extracted = Extract part... 0 '.newduration' rectangular 1 no

	if extra_VOT_silence_buffer > 0
		select Sound extra_VOT_silence_buffer
		plus extracted
		Concatenate
		Rename... VOT_aspiration_'.number'
	else
		select extracted
		Copy... VOT_aspiration_'.number'
	endif

	select extracted
	Remove
endproc


procedure make_prevoicing_ind .prevoicing_dur .sound_to_extract_from .onset_point .f0_step_index .newname$
	#>> Create a faux "prevoicing" segment
	#>> to preappend to sounds whose VOT is negative.
	#>> This just extracts the onset of the vowel, 
	#>> shapes the amplitude envelope,
	#>> and low-pass filters so it sounds like a murmur. 
	#>> It also matches the F0 to the onset F0 of the vowel 
	#>> to which it will be attached. 

	# Extract with trianglar duration
		select .sound_to_extract_from
		Copy... temp_to_extract_from
		To Manipulation: 0.01, 75, 600
		Extract pitch tier
		Remove points between: 0, 25
	# add a single point where the F0 will be
	# at the start of the vowel 
		temp_F0_value = step_F0_'.f0_step_index'
		Add point: 0.01, temp_F0_value
		selectObject: "Manipulation temp_to_extract_from"
		plusObject: "PitchTier temp_to_extract_from"
		Replace pitch tier
		selectObject: "Manipulation temp_to_extract_from"
		Get resynthesis (overlap-add)
		Rename... temp_to_extract_from_pitch_matched

		Extract part: .onset_point, .onset_point + .prevoicing_dur, "triangular", 1, "no"
		Rename... junk
		Filter (pass Hann band): 0, 500, 400
	# Attenuate
		Multiply... 0.4
		Rename... '.newname$'

	# cleanup
		select Sound junk
		plus Sound temp_to_extract_from
		plus Manipulation temp_to_extract_from
		plus PitchTier temp_to_extract_from
		plus Sound temp_to_extract_from_pitch_matched
		Remove
endproc


procedure make_silent_buffer
	#>> Creates silence to buffer onset & offset,
	#>> where F0 mis-estimations can happen
	Create Sound from formula: "buffer", 1, 0, 1, samplerate, "0"
endproc


procedure create_F0_manipulation_object
	#>> Wrapper for creating the object that you can manipulate
	#>> in terms of duration & F0
		select Sound 'voiced$'
		To Manipulation... 0.01 minpitch maxpitch
endproc


procedure make_VOT_cutbacks
	#>> Takes the vowel, and progressively cuts away the onset,
	#>> which will be replaced by the aspiration noise.
	
	# Loop through the continuum indices
	for vot_step_index from 1 to number_of_VOT_steps

	# Set burst duration; 
	# This allows up to a certain VOT without any cutback at all. 
		minimum_burst_dur = 0.008
		vot_relative_to_burst = step_VOT_'vot_step_index' - minimum_burst_dur
	
	# Calculate cutback duration
	# as a product of the VOT and the cutback ratio
		cutback_dur_'vot_step_index' = vot_relative_to_burst * vowel_cutback_to_VOT_ratio

	# If the cutback duration exceeds the max allowed,
	# just set it to be the max allowed
		if cutback_dur_'vot_step_index' > max_cutback_dur
			cutback_dur_'vot_step_index' = max_cutback_dur
		endif

	# If this step is prevoiced,
	# set cutback to be zero
		if step_VOT_'vot_step_index' < minimum_burst_dur
			cutback_dur_'vot_step_index' = 0
		endif

	# Set the point where the vowel begins
	# (i.e. the cutback point)
	# Relative to vowel onset that the user selected. 
		select Sound 'voiced$'
		.dur = Get total duration
		cutpoint_start = vowel_start_point + cutback_dur_'vot_step_index' - (vot_crossfade_duration/2)

	# Extract the vowel cutback portion for this step
		Extract part: cutpoint_start, .dur, "rectangular", 1, "no"
		Rename... vowel_cutback_'vot_step_index'
	endfor
endproc


procedure modify_and_assemble_components_and_save
	#>> this is the function that loops through and performs 
	#>> all the sequential actions needed to build the continuum,
	#>> including F0 manipulation,
	#>> and concatenation of aspiration & vowel. 
	
    	# Loop through the continuum indices
	for vot_step_index from 1 to number_of_VOT_steps
		#----------------------------------------------------#
		# If you chose to covary VOT and F0,
		# then set the F0 step to be the same as the VOT step,
		# and only run a single F0 step (not a whole loop of them
		# Note how there is no for-loop started in this part,
		# unlike for the following section)
		if covary = 1
			current_F0_value = step_F0_'vot_step_index'
			current_F0_step = 'vot_step_index'

			call manipulate_vowel_f0_onset
			call put_all_pieces_together_and_save
			
			# Cleanup for each step of the VOT continuum
				selectObject: "Manipulation temp_cutback_'vot_step_index'"
				plusObject: "PitchTier temp_cutback_'vot_step_index'"
				plusObject: "Sound temp_cutback_'vot_step_index'"
				plusObject: "Sound vowel_cutback_'vot_step_index'"
				Remove
			
		elsif covary = 0
			# If you are setting VOT and F0 independently
			# (in a crossed-cue matrix of sounds),
			# begin a new nested loop through all the F0 steps here
			
			for current_F0_step from 1 to number_of_f0_steps
				current_F0_value = step_F0_'current_F0_step'
				call manipulate_vowel_f0_onset
				call put_all_pieces_together_and_save
				
				selectObject: "Manipulation temp_cutback_'vot_step_index'"
				plusObject: "Sound temp_cutback_'vot_step_index'"
				plusObject: "PitchTier temp_cutback_'vot_step_index'"
				Remove
			endfor
			selectObject: "Sound vowel_cutback_'vot_step_index'"
			Remove
		endif
		#----------------------------------------------------#
	# remove the aspiration segment
	select Sound VOT_aspiration_'vot_step_index'
	Remove
	
	endfor
	# end loop through continuum indices
	
	select Sound buffer
	plus Manipulation 'voiced$'
	if extra_VOT_silence_buffer > 0
	plus Sound extra_VOT_silence_buffer
	endif
	Remove
endproc

procedure check_for_clean_up_stimuli
	#>> If the script is about to produce 25+ sounds,
	#>> and if they are being saved,
	#>> then ask the user if they want them to be cleaned up
	#>> from the object list along the way
	#
	
	beginPause: "Set the target directory"
	    comment: "You are about to create 'total_number_of_stimuli' sounds."
	    comment: "Do you want to keep them in the object list?"
	    comment: "Or do you want to empty the object list as it rolls along?"
	clicked = endPause: "Keep sounds in the list", "Remove sounds from the list", 1
	
	if clicked == 1
	    clean_up_stimuli_in_list = 0
	elsif clicked == 2
	    clean_up_stimuli_in_list = 1
	endif
endproc


procedure manipulate_vowel_f0_onset
	#>> Manipulate the pitch of the vowel onset
	#>> as it typically covaries with VOT
	#
	#>> This is called within a loop through all the continuum indices,
	#>> Hence it inherits 'vot_step_index'
	#>> and also inherits 'current_F0_value' 
	#>> from a loop in the parent procedure  'modify_and_assemble_components',	
	#----------------------------------------------------#
	# Add the silent buffer at the onset
	# so that F0 tracking isn't lost at the edge of the signal
		selectObject: "Sound buffer"
		plusObject: "Sound vowel_cutback_'vot_step_index'"
		Concatenate
		Rename: "temp_cutback_'vot_step_index'"
		To Manipulation: 0.01, minpitch, maxpitch
		Extract pitch tier
		Remove points between: 0, 1 + f0duration
		# add pitch point 10 ms into the vowel 
		# (note the 1 second addded)
		Add point: 1.01, current_F0_value
		selectObject: "Manipulation temp_cutback_'vot_step_index'"
		plusObject: "PitchTier temp_cutback_'vot_step_index'"
		Replace pitch tier
	#----------------------------------------------------#
	# Create the new sound with the new pitch contour
		selectObject: "Manipulation temp_cutback_'vot_step_index'"
		Get resynthesis (overlap-add)

	# Arbitrary rename, 
	# because we're just going to extract only a piece out of it
		Rename... junk
		.dur = Get total duration
	#----------------------------------------------------#
	# Extract the part without the 1-s buffer
		Extract part: 1, .dur, "rectangular", 1, "no"
		Rename: "vowel_cutback_'vot_step_index'_f0_'current_F0_step'"
		.sound_to_shape_onset = selected("Sound")

	#----------------------------------------------------#
	# Shape the vowel onset intensity envelope, if desired
		if shape_vowel_envelope_onset == 1
			ramp_duration = cutback_dur_'vot_step_index'/2
			call half_fade_in_by_index .sound_to_shape_onset ramp_duration
			
			print Step 'vot_step_index' half-ramped vowel onset intensity over the first 'ramp_duration:3' s'newline$'
		endif
	#----------------------------------------------------#
	# Clean up the sound that had the silent buffer
		select Sound junk
		Remove
endproc


procedure half_fade_in_by_index .sound .absolute_fade_in_duration
	#>> Applied an onset ramp
	#>> that grows from 1/2 intensity to full intensity
	#>> in a cosine-ramp shape. 
	#>> Note: it doesn't copy into a new object,
	#>> but rather modifiies the selected object in place. 
		select .sound
		.start = Get start time
			Formula... if x < ('.absolute_fade_in_duration')  
		...then self * ( 1 + cos(( x - ('.absolute_fade_in_duration' - '.start')) / '.absolute_fade_in_duration' * pi /2 )) / 2  
		...else self endif  
endproc


procedure put_all_pieces_together_and_save
	#>> Pre-appends the VOT aspiration,
	#>> Names them appropriately,
	#>> and saves the files.
        
        call concatenate_asp_and_vowel
        call rename_objects

        # Save the file, if desired
        if save_files > 1
        	Save as WAV file: parent_folder$ + "/" + new_output_folder$ + "/" + "Stimuli" + "/" + "'basename$''f0Prefix$''current_F0_step$''votPrefix$''vot_step_index$'.wav"
        	
        	if clean_up_stimuli_in_list == 1
        		Remove
        	endif
        endif
endproc


procedure concatenate_asp_and_vowel
	#>> Concatenate aspration and vowel
	#>> Note that they are crossfaded, not striahgt concatenated
	
	select Sound VOT_aspiration_'vot_step_index'
	plus Sound vowel_cutback_'vot_step_index'_f0_'current_F0_step'
	Concatenate with overlap... vot_crossfade_duration

	# This part is NOT run by default
	if scale_to_equal_RMS$ == "yes"
		Scale intensity... equal_RMS_intensity_target
	endif
	
	# Remove the vowel portion, 
	# because it's only used in this procedure
	# Don't remove the aspiration,
	# because it might be used for other vowel portions
	# that have different F0
	select Sound vowel_cutback_'vot_step_index'_f0_'current_F0_step'
	Remove
	
	select Sound chain
endproc


procedure rename_objects
	#>> make the names pretty
	#>> by adding leading zeros in the case of 10+ step continua
	#>> and using user-specified base names & prefixes
	
	# Establish formatted continuum step numbers
		call add_leading_zeros

	# Select sound object that was just concatenated
		select Sound chain

	# Assign the name based on the formatted numbers
		Rename... 'basename$''f0Prefix$''current_F0_step$''votPrefix$''vot_step_index$'
		# Make a clean copy of the object
		# to prevent Praat crash when concatenating
		call pseudo_copy
endproc


procedure add_leading_zeros
	#>> Add leading zeros before single digits
	#>> if this is a double-digit-member VOT continuum
	#>> so instead of  7,  8,  9, 10, 11,
	#>> it's          07, 08, 09, 10, 11.
	#>> This is helpful for alphabetic sorting,
	#>> Where you could get 1, 10, 11, 12... 19, 2, 20...
			
	# first declare named digits
	vot_step_index$ = "'vot_step_index'"
	current_F0_step$ = "'current_F0_step'"

	# Additionally ... 
	# if there's only one F0 step,
	# then omit the F0 prefix name altogether
	if number_of_f0_steps < 2
		# get rid of the F0 prefix if there is only one F0 step
		tempf0Prefix$ = ""
		current_F0_step$ = ""
	endif

	if format_fixed_digits = 1
		# if there are more than 9 steps,
		if number_of_VOT_steps > 9
				# if it's  single-digit index,
				if 'vot_step_index' < 10
					# add a leading 0 to the index name	
					vot_step_index$ = "0'vot_step_index'"
				endif
		endif

		# do the same for the F0 steps
		if number_of_f0_steps > 9
			if 'current_F0_step' < 10
				current_F0_step$ = "0'current_F0_step'"
			endif
		endif
	endif
endproc


procedure pseudo_copy
	#>> Copies a sound into a new sound object
	#>> It seems like a waste of computation,
	#>> but it avoids  Praat crash that happens
	#>> when you try to concatenate sounds 
	#>> produced by any loop process
	
	temp_name$ = selected$("Sound")
	end = Get end time
	Extract part: 0, end, "rectangular", 1, "no"
	Rename... temp_to_rename
	select Sound 'temp_name$'
	Remove
	select Sound temp_to_rename
	Rename... 'temp_name$'
endproc	


procedure assemble_continuum_series_to_view
	#>> if the continuum is under 10 steps, call it up in a display
	if number_of_VOT_steps < 10
		if covary = 1
			selectObject: "Sound 'basename$''f0Prefix$'1'votPrefix$'1"
			for continuum_step from 2 to number_of_VOT_steps
				plusObject: "Sound 'basename$''f0Prefix$''continuum_step''votPrefix$''continuum_step'"
			endfor
			
			Concatenate recoverably
			select Sound chain
			Rename... VOT_continuum
			select TextGrid chain
			Rename... VOT_continuum
			
			select Sound VOT_continuum
			plus TextGrid VOT_continuum
			View & Edit
		endif
	endif
endproc


procedure save_info_files
	#>> Save the stimuli, continuum info, file list and original sounds
	#>> ... only if you chose to save the files

	if save_files > 1
		## Save the continuum info numbers in your folder
		call save_info_window "'parent_folder$'/'new_output_folder$'" 'output_filename$'

		## Save the original sounds			
			select voiced_sound_index
			Save as WAV file: "'parent_folder$'/'new_output_folder$'/Original_sounds/" + voiced$ + ".wav"
			
			select voiceless_sound_index
			Save as WAV file: "'parent_folder$'/'new_output_folder$'/Original_sounds/" + voiceless$ + ".wav"

			select full_aspiration_sound
			Save as WAV file: "'parent_folder$'/'new_output_folder$'/Original_sounds/" + full_aspiration_sound$ + "_used_for_concatenation.wav"
			
			select full_aspiration_sound
			Save as WAV file: "'parent_folder$'/'new_output_folder$'/Original_sounds/" + full_aspiration_sound$ + "_used_for_concatenation.wav"

		## Make a list of all the files that you made
			call make_file_list "'parent_folder$'/'new_output_folder$'/Stimuli" 'listName$'
	endif
endproc


procedure save_info_window output_folder$ output_filename$
	#>> Takes the contents of the info window,
	#>> and saves it to a text file for later inspection
	
	if save_files > 1
		filedelete 'output_folder$'/'output_filename$'.txt
		fappendinfo 'output_folder$'/'output_filename$'.txt
endproc


procedure make_file_list .soundDir$ listName$
	#>> Make a list of all the files in a directory
	#>> (e.g. for making a sitmulus list)
		Create Strings as file list... 'listName$' '.soundDir$'/*.wav
		Save as raw text file... 'parent_folder$'/'new_output_folder$'/'listName$'.txt
		select Strings 'listName$'
		Remove
endproc	


procedure cleanup_remaining
	# the objects to remove are the aspiration segments 
	# that were extracted from the sounds. 
	# only do this if you didn't use a pre-arranged aspiration,
	# because if you did, 
	# you probably still want it in the list. 
	
	if preselected == 0
		select full_aspiration_sound
		plus original_aspiration_sound
		Remove
	endif
endproc

procedure set_variables_at_startup
	#>> Convert variable names from the startup form
	#>> and declare a bunch more variables
	#>> that currently have reasonable defaults.

	############################################
	# NAMES
		# Main name prefix for the objects
			# stimulus base filename
			basename$ = "'sound_object_name_prefix$'"
			# main sub-folder in the parent directory
			# where your continuum will be saved

		# Ensure that the parent folder exists,
		# and that the current cub-folder name 
		# doesn't already exist from a previous run
		# of the script. 
		# if everything is clear, make the new sub-directories. 
			if save_files > 1
				call check_if_parent_folder_exists 'parent_folder$'
				call check_if_subfolders_already_exist
				call make_save_directories
			endif

	############################################
	# BASIC CONTINUUM INFO
		# VOT range
		# pull the startup window variables
		# Convert ms to seconds
			vot_max = right_vot_min_and_max/1000
			vot_min = left_vot_min_and_max/1000

		# Number of F0 continuum steps
			number_of_f0_steps = f0_steps
			min_F0_onset = left_min_and_max_F0_onset
			max_F0_onset = right_min_and_max_F0_onset

		# Style of F0 manipulation (absolute or relative)
			if f0_manipulation == 1
			    f0_manipulation_style$ = "relative_change_from_original"
			elsif f0_manipulation == 2
			    f0_manipulation_style$ = "absolute_Hz"
			else
			    exit f0_manipulation_style not supported. Check input parameters.
			endif

			if f0_manipulation_style$ == "absolute_Hz"
			    if min_F0_onset < 50
				pause You selected a starting F0 of 'min_F0_onset' absolute Hz. Did you mean to set F0 change relative to original F0?
			    endif
			endif

		# Convert F0 perturbation time range from ms to seconds 
			f0duration = (f0_time_range_in_ms/1000)

	############################################
	# COVARIANCE of VOT and F0
	# If you want to co-vary the F0 along with the VOT,
	# then the F0 continuum needs to have 
	# the same number of steps as the VOT continnuum
	# first convert it from a 1-2 to a 0/1
    	# first convert name from startup menu
   		 covary = vOT_and_F0_covariance
    	
    	# since the choices are 1 and 2, convert to 0 and 1
		covary = covary - 1
		if covary = 1
			# If VOT and F0 are covaried,
			# then override the number of F0 steps 
			# to be the same as the number of VOT steps
			number_of_f0_steps = number_of_VOT_steps
		endif

	# Check to see if the user wants to clean up stimuli
	# as they're being created,
	# if they're going to be saved anyway
	# (this is useful when you're making a *lot* of stimuli)
	# Default value is to not clean up
	clean_up_stimuli_in_list = 0
	
		if covary = 1
			total_number_of_stimuli = number_of_VOT_steps
		elsif covary = 0
			total_number_of_stimuli = number_of_VOT_steps * number_of_f0_steps
		endif

		if total_number_of_stimuli  > 24
			if save_files > 1
				call check_for_clean_up_stimuli
			endif
		endif

	# Covary intensity along with VOT... set at startup
	# Apply this also for bursts in prevoiced sounds?
		covary_even_for_prevoicing = 0

	############################################
	# Logical value that tells whether the user already has pre-selected 
	# burst & aspiration objects
		preselected = 'method_of_choosing_aspiration' - 1
	# if so, this value is 1. 
	# In typical circumstances, this value will be 0,
	# because the user will select the aspiration from an object
	# that contains a whole syllable. 
	# we're subtracting 1 because the input isn't 0/1; it's 1/2
	# corresponding to the 1st or 2nd choice from the option menu. 

	############################################
	# Set some hard-coded variables
	# basic booleans
		yes = 1
		no = 0

	############################################
	# NAMES
		# Intra-filename continuum variable level prefixes
		votPrefix$ = "_VOT_"	
		f0Prefix$ = "_F0_"

		# Sets output (Continuum info) file name using the basename from the folder
		output_filename$ = "'basename$'_Continuum_Info"

		# Sets name for the file list using the basename from the folder
		listName$ = "'basename$'_file_list"

		# Ensure that single-digit numbers have a leading 0 
		# to align them in character space
		format_fixed_digits = 1
		
	############################################
	# PITCH analysis settings - 
	# change if you have a super-low or super-high frequency talker
		minpitch = 65
		maxpitch = 275
		
	############################################
	# Re-scaling the RMS intensity of the stimuli
	# I do NOT recommend setting this to "yes", 
		# it only equalizes the mathematical values across the continuum,
		# which **should** be different, given that aspiration
		# is lower in intensity than voiced speech)
		# if you equalize RMS, you might think you're equalizing loudness,
		# but you're really just introducing a loudness confound
		# (as this can make stimuli with longer VOTs unnaturally louder)
	
		scale_to_equal_RMS$ = "no"
		
		# but if you must... 
		equal_RMS_intensity_target = 70

	############################################
	# RISE-TIME for the cutback vowel
		# The original (cutback) vowel has the following pseudo-risetime applied to it
		# over this duration, a half-cosine fade will be applied, 
		# so that it goes from *half* amplitude to *full* amplitude
		# in this time interval. 
		# This makes the voiceless items smoother 
		# by adding an amplitude pinch at the segment boundary.
		# Otherwise, you have this unnatural sequence of soft-intensity aspiration
		# followed immediately by a full-intensity vowel with rapid onset.
		# This could cause backward masking of the aspiration
		# Shapes the vowel onset to half-attenuate it at onset (1 = yes / 0 = no)
		##
		###
		###### -> 
			shape_vowel_envelope_onset = 1

	# Sets an absolute buffer duration for silence before the sound file
		##
		###
		###### -> 
			extra_VOT_silence_buffer = 0.05
			
	############################################
	# FINE CONTROL in the manipulation window? (advanced users only)
	# this calls up the aspirations for you to edit if you want. 
		#  A good option if the pitch in your final stimuli don't seem to be correct,
		#  because you can see whether the pitch was correctly tracked to start with.
		#  - usually it means that you have incorrect pitch analysis settings 
		#    (min & max pitch range), but you can override this
		#     if you are especially skilled in the Manipulation window)
		##
		###
		###### -> 
			extracontrol = 0
		
	############################################
	# CROSS-FADING aspiration into the vowel
		# Sets a window across which the VOT will be extended as it fades into the vowel
		# for example, 0.01 means that the aspiration will go 5ms past the "VOT" value 
		# and that extension will blend into the vowel onset to create a smooth transition
		# NOTE: the calculated VOT values do not take this adjustment into account,
		# so if you set it to a value above 0.001, you'll want to subtract half the value
		# from every VOT step value. 
		##
		###
		###### -> 
			vot_crossfade_duration = 0.006
		
	############################################
	# CUTBACK RATIO and cutback LIMITS
		# Sets a ratio of VOT to vowel cutback duration. 
		# If this value is 0.5, then for every 2ms of VOT, 1ms of vowel is cutback. 
		# a value of 1 means that for each ms of VOT, 
		# there is a corresponding ms of vowel cutback. 
		##
		###
		###### -> 
			vowel_cutback_to_VOT_ratio = 0.65
		
		# Maximum duration of vowel to cut back
		# This would prevent an entire short vowel from being eliminated
		# e.g. the vowel in "bit"
		##
		###
		###### -> 
			max_cutback_dur = 0.7
endproc

