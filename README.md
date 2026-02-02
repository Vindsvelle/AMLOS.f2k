# AMLOS.f2k - Automated Music Library Organization Scripts (for foobar2000)
My personally developed (and hideously inelegant & unoptimized) scripts for automated music library organization via foobar2000's "File Operations" tool. Feel free to request a custom script!

## Provisional / placeholder Readme
_Since I really need to learn GH markup standards before this thing's half-legible (let alone **useful** to anyone other than me) consider this a draft._

## Purpose
Automated, user-customized music library organization via foobar2000's "File Operations" tool

## Function
Using an AMLOS script in F2K's _File Operations_ auto-magically does the following:
1. Generates directory structure (creates root dir(s) if absent; creates album dir(s))
2. Renames source track files
3. Relocates source track files to the newly generated album dir
4. Relocates other album resources (e.g. external album art, .nfo files, etc.) to new album dir
5. Checks if source dir(s) are empty and, if so, deletes them

 ### _**Includes contingencies for:**_
  * many commonly-used codecs, irrespective of lossless / lossy (I chiefly favor FLAC and OPUS for those two respective encoding types, but it _(should)_ cover all the usual suspects, as well as niche stuff like DSD)
  * files with atypical properties, e.g. multi-channel (≥3 channel)
  * %BANDCOUNTRY% tag (a temporary and wildly indelicate workaround I use in cases of artist name ambiguity, wherein I create the metadata field %BANDCOUNTRY% and assign the 3-letter ISO code of the band's country of origin -- and, if still _further_ distinguishment is needed, follow it with their city. _I need a better solution._)
  * compilation / "various artist" albums, etc.
  * articles in artist names ("The","A", etc.; a function baked into Title Formatting itself)

## Base Organizational Schema
_(Note: backslash character \ = new directory; curly brackets {} = conditionals)_  

<code>CODEC \ ALBUM ARTIST{ '(</code>three-letter country code<code>)'} - [DATE{ '(YEAR Reissue)'}] ALBUM (@CODEC{ / </code>BITRATE conditionals for lossy & lossless, e.g. <code>@FLAC / @320k MP3 / @DSD64}) \ {DISCNUMBER.</code> if ≥2<code>}TRACKNUMBER. TITLE</code>  

**Example Output:** <code>FLAC\Absu - [2001] Tara (@FLAC)\03. A Shield With An Iron Face.flac</code>

## AMLOS

### Current Standard Script
<code>$puts(D4,$substr($meta(date),1,4))$puts(ORD,$substr($meta(original release date),1,4))$puts(CP,%codec_profile%)$if($stricmp($substr(%codec%,1,3),DSD),DSD,$upper(%codec%))\$swapprefix(%album artist%)$put(bc,[$if($meta_test(bandcountry), '('$meta(bandcountry)')')]) - $if($stricmp(%album artist%,Various Artists),%album%[ $if($get(ORD),$if($stricmp($get(ORD),$get(D4)),'['$get(ORD)']','['$get(ORD) '('$get(D4) reissue')'']'),'['$get(D4)']')],[$if($get(ORD),$if($stricmp($get(ORD),$get(D4)),'['$get(ORD)']','['$get(ORD) '('$get(D4) reissue')'']'),'['$get(D4)']') ]%album%) [$if($stricmp($substr($get(CP),1,3),VBR),'('@$get(CP) %codec%')',)][$if($stricmp($get(CP),CBR),'('@%bitrate%k %codec%')',)]$if($stricmp($substr(%codec%,1,4),FLAC),$ifequal(%__bitspersample%,16,'('@FLAC')','('@FLAC %__bitspersample%bit $iflonger(%samplerate%,5,$left(%samplerate%,3),$left(%samplerate%,2))kHz$ifgreater(%channels%,2,',' %channels%,)')'),)[$if($stricmp($get(CP),ABR),'('@$get(CP)')',)][$if($stricmp(%codec%,AAC),'('@%codec%')',)][$if($stricmp(%codec%,ALAC),'('@%codec%')',)][$if($stricmp(%codec%,OPUS),'('@$upper(%codec%)')',)]$if($stricmp($substr(%codec%,1,3),DSD),'('@%codec%')',)\$if(%tracknumber%,$ifgreater(%totaldiscs%,1,%discnumber%.%tracknumber%,%tracknumber%). ,)$if($not($stricmp(%album artist%,%artist%)),%artist%$get(bc) - ,)%title%[$if($stricmp(%codec%,AAC), '{'%bitrate%kbps'}',)]</code>

### No Codec Profile
<code>$puts(D4,$substr($meta(date),1,4))$puts(ORD,$substr($meta(original release date),1,4))$puts(CP,%codec_profile%)%codec%\$swapprefix(%album artist%)$put(bc,[$if($meta_test(bandcountry), '('$meta(bandcountry)')')]) - $if($stricmp(%album artist%,Various Artists),%album%[ $if($get(ORD),$if($stricmp($get(ORD),$get(D4)),'['$get(ORD)']','['$get(ORD) '('$get(D4) reissue')'']'),'['$get(D4)']')],[$if($get(ORD),$if($stricmp($get(ORD),$get(D4)),'['$get(ORD)']','['$get(ORD) '('$get(D4) reissue')'']'),'['$get(D4)']') ]%album%) '('@%codec%')'\$if(%tracknumber%,$ifgreater(%totaldiscs%,1,%discnumber%.%tracknumber%,%tracknumber%). ,)$if($not($stricmp(%album artist%,%artist%)),%artist%$get(bc) - ,)%title%</code>

### Album {@codec, bitdepth, bitrate, %CHANNELS% appended to filename}
<code>$puts(D4,$substr($meta(date),1,4))$puts(ORD,$substr($meta(original release date),1,4))$puts(CP,%codec_profile%)%codec%\$swapprefix(%album artist%)$put(bc,[$if($meta_test(bandcountry), '('$meta(bandcountry)')')]) - $if($stricmp(%album artist%,Various Artists),%album%[ $if($get(ORD),$if($stricmp($get(ORD),$get(D4)),'['$get(ORD)']','['$get(ORD) '('$get(D4) reissue')'']'),'['$get(D4)']')],[$if($get(ORD),$if($stricmp($get(ORD),$get(D4)),'['$get(ORD)']','['$get(ORD) '('$get(D4) reissue')'']'),'['$get(D4)']') ]%album%) [$if($stricmp($substr($get(CP),1,3),VBR),'('@$get(CP)')',)][$if($stricmp($get(CP),CBR),'('@%bitrate%k')',)]$if($stricmp($substr(%codec%,1,4),FLAC),$ifequal(%__bitspersample%,16,'('@FLAC')','('@FLAC %__bitspersample%bit $iflonger(%samplerate%,5,$left(%samplerate%,3),$left(%samplerate%,2))kHz')'),)[$if($stricmp($get(CP),ABR),'('@$get(CP)')',)][$if($stricmp(%codec%,AAC),'('@%codec%')',)][$if($stricmp(%codec%,ALAC),'('@%codec%')',)][$if($stricmp(%codec%,AAC), '('%bitrate%kbps')',)]\$if(%tracknumber%,$ifgreater(%totaldiscs%,1,%discnumber%.%tracknumber%,%tracknumber%). ,)$if($not($stricmp(%album artist%,%artist%)),%artist%$get(bc) - ,)%title% [$if($stricmp($substr($get(CP),1,3),VBR),'{'@$get(CP)'}',)][$if($stricmp($get(CP),CBR),'{'@%bitrate%k'}',)]$if($stricmp($substr(%codec%,1,4),FLAC),$ifequal(%__bitspersample%,16,'{'@FLAC'}','{'@FLAC %__bitspersample%bit $iflonger(%samplerate%,5,$left(%samplerate%,3),$left(%samplerate%,2))kHz$ifgreater(%channels%,2,',' %channels%,)'}'),)[$if($stricmp($get(CP),ABR),'{'@$get(CP)'}',)][$if($stricmp(%codec%,AAC),'{'@%codec%'}',)][$if($stricmp(%codec%,ALAC),'{'@%codec%'}',)][$if($stricmp(%codec%,AAC), '{'%bitrate%kbps'}',)]</code>
