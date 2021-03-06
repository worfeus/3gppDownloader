<!-- omit in toc -->
# Download doc from 3GPP

<!-- Download 3GPP pdf files from http://www.etsi.org/deliver/etsi_ts/xxx -->
Download 3GPP zip(doc) files from *<http://www.3gpp.org/ftp//Specs/archive>*

Conversion from doc to pdf needs **MS Word installed under Win OS**

```shell
usage: 3gpp download [-h] [-a | -d | -c] [-m MULTITHREAD] [-r RELEASE [RELEASE ...]] [-s SERIES [SERIES ...]] [-p PATH]

optional arguments:
  -h, --help            show this help message and exit
  -a, --all             Perform Download, Extract and Convert
  -d, --download        Perform Download only
  -c, --convert         Perform Convert word to pdf (win+word only)
  -m MULTITHREAD, --multithread MULTITHREAD
                        Multi-thread used, the default is 8
  -r RELEASE [RELEASE ...], --release RELEASE [RELEASE ...]
                        Indicate the 3GPP release number, default is 15 (LTE: 8+, NR: 15+)
  -s SERIES [SERIES ...], --series SERIES [SERIES ...]
                        Download the 3GPP series number, eg. 36 for EUTRAN, 38 for NR
  -p PATH, --path PATH  Saving path for downloaded 3GPP
```

- [Work Mode](#work-mode)
  - [Download Only `-d`](#download-only--d)
  - [Convert Only `-c`](#convert-only--c)
  - [Download and Convert `-a`](#download-and-convert--a)
- [Download](#download)
  - [Specific output folder](#specific-output-folder)
  - [Specific release and series](#specific-release-and-series)
- [Convert](#convert)
  - [Convert to pdf](#convert-to-pdf)
  - [Compatible Issues](#compatible-issues)
  - [Merge](#merge)
    - [Sejda](#sejda)
      - [Suppress Log levels](#suppress-log-levels)
    - [Pypdf2](#pypdf2)
      - [Encoding Issues](#encoding-issues)

## Work Mode

```shell
# download 36 series, latest version of relase 15
# save reports under './36Series_Rel15/xxx.doc'
python 3gppDownloader.py -d -r 15 -s 36


# download both 36 and 38 series, latest version of relase 15
# save reports under './36Series_Rel15/xxx.doc' and './38Series_Rel15/xxx.doc'

# only perform convert doc to pdf (only works under windows with Word installed)
python 3gppDownloader.py -c -r 15 -s 36 38


# download both 36 and 38 series, latest version of relase 15
# save reports under './36Series_Rel15/xxx.doc', './38Series_Rel15/xxx.doc', 
# './36Series_Rel16/xxx.doc', './38Series_Rel16/xxx.doc'

# both download and convert format
python 3gppDownloader.py -a -r 15 16 -s 36
```

### Download Only `-d`

**Download** and **extract** zip files from *<http://www.3gpp.org/ftp//Specs/archive>* according input release and series numbers

### Convert Only `-c`

**Convert** doc/docx files to pdf and **merge** split pdf if needed

- Convert function requires **Microsoft Word** installed under **windows** system
- Merge function requires **sejda + java** or **pypdf2** python module installed

### Download and Convert `-a`

Perform both download and conversion

## Download

### Specific output folder

```shell
# spec download and save as .\36Series_Rel15\xxx.doc
python 3gppDownloader.py -d -r 15 -s 36

# spec download and save as /user/xx/Desktop/36Series_Rel15/xxx.doc
python 3gppDownloader.py -d -r 15 -s 36 -p /user/xx/Desktop/
```

### Specific release and series

The series(-s) and release(-r) could accept multiple numbers

```shell
# download 36 series, latest version of relase 15
python 3gppDownloader.py -d -r 15 -s 36

# download both 36 series release 15 and 38 series relase 15
python 3gppDownloader.py -d -r 15 -s 36 38

# download 36, 38 series, both release 15 and 16
python 3gppDownloader.py -d -r 15 16 -s 36 38
```

## Convert

### Convert to pdf

To convert word to pdf, **Microsoft Word** is required under **windows**

```powershell
# only perform convert for reports under .\36Series_Rel15\xxx.doc
python 3gppDownloader.py -c -r 15 -s 36
```

### Compatible Issues

Sometimes the MS Word is no response for long time, and conversion fails due to compatible issues, suggest to:

- Run conversion mode multiple times if needed
- Terminate word processes before conversion `Stop-Process -Name "winword"` (powershell)

Very few documents, like 36440-f00.doc, 36523-3-g40_s00-s06.doc, cannot **save as** pdf in MS Word:

- Manually convert doc to pdf using **print** function
- Add bookmarks to pdf generated by print function

### Merge

Some report may split into multiple doc, after conversion, like 36101-fa0 report, which consists of:

- 36101-fa0_cover.pdf
- 36101-fa0_s00-07.pdf
- 36101-fa0_s08-XX.pdf
- 36101-fa0_sAnnexes.pdf

Hence, additional steps performed to merge all above reports into *36101-fa0.pdf*

#### Sejda

A java based api console, download **sejda-console-x.x.xx-bin.zip**
 at [here](https://github.com/torakiki/sejda/releases/tag/v3.2.85)

Extract and make sure **sejda-console.bat** available at **./bin/sejda-console.bat**

> |----3gppDownloader.py
> |----bin
> |     |----sejda-console
> |     |----sejda-console.bat
> |----lib
> |     |----xxxx.jar
> |     |----xxxx.jar
> |----etc
> |     |----logback.xml
> |     |----SEJDA_LICENSE.txt

##### Suppress Log levels

By default, there is massive logs outputs in the console, adjust log level in `etc/logback.xml` to suppress if needed

```xml
	<logger name="org.sejda" level="ERROR" />
	<logger name="org.sejda.sambox" level="ERROR" />
	<logger name="org.hibernate.validator" level="ERROR" />

	<root level="ERROR">
```


#### Pypdf2

If sejda not detected under code folder, pypdf2 python module will be used

##### Encoding Issues

If following errors promoted during pypdf2 merging:

> File "3gppDownloader\.env\lib\site-packages\PyPDF2\utils.py", line 238, in b_
>     r = s.encode('latin-1')
> UnicodeEncodeError: 'latin-1' codec can't encode characters in position 8-9: ordinal not in range(256)


An quick alternative to suppress this error is to made following changes to "PyPDF2\utils.py"

```python
	def b_(s):
        bc = B_CACHE
        if s in bc:
            return bc[s]
        if type(s) == bytes:
            return s
        else:
            # line 238 at PyPDF2/utils.py
            # modifiction start
            # r = s.encode('latin-1')
            try:
                r = s.encode('latin-1')
            except:
                r = s.encode('utf-8')
            # modification end
            if len(s) < 2:
                bc[s] = r
            return r

```

