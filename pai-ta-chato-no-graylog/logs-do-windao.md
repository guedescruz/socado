# Logs do Windao

Download do nxlog + sysmon + ativa os logs do firewall

{% embed url="https://rubular.com/" %}



```text
Panic Soft
#NoFreeOnExit TRUE

define ROOT     C:\Program Files (x86)\nxlog
define CERTDIR  %ROOT%\cert
define CONFDIR  %ROOT%\conf
define LOGDIR   %ROOT%\data
define LOGFILE  %LOGDIR%\nxlog.log
LogFile %LOGFILE%

Moduledir %ROOT%\modules
CacheDir  %ROOT%\data
Pidfile   %ROOT%\data\nxlog.pid
SpoolDir  %ROOT%\data

<Extension _syslog>
    Module      xm_syslog
</Extension>

<Extension _charconv>
    Module      xm_charconv
    AutodetectCharsets iso8859-2, utf-8, utf-16, utf-32
</Extension>

<Extension _exec>
    Module      xm_exec
</Extension>

<Extension gelf>
    Module        xm_gelf
</Extension>

<Extension _fileop>
    Module      xm_fileop

    # Check the size of our log file hourly, rotate if larger than 5MB
    <Schedule>
        Every   1 hour
        Exec    if (file_exists('%LOGFILE%') and \
                   (file_size('%LOGFILE%') >= 5M)) \
                    file_cycle('%LOGFILE%', 8);
    </Schedule>

    # Rotate our log file every week on Sunday at midnight
    <Schedule>
        When    @weekly
        Exec    if file_exists('%LOGFILE%') file_cycle('%LOGFILE%', 8);
    </Schedule>
</Extension>

<Input sysmon>
	Module im_msvistalog
	<QueryXML>
		<QueryList>
			<Query Id='0'>
				<Select Path='Microsoft-Windows-Sysmon/Operational'>*</Select>
			</Query>
		</QueryList>
	</QueryXML>
</Input>

<Input fwlog>
	Module	im_file
	File	'C:\windows\system32\LogFiles\Firewall\pfirewall.log'
   Exec		if $raw_event =~ /(\d\d\d\d\-\d\d-\d\d \d\d:\d\d:\d\d)/ {$EventTime = strptime($1, '%Y-%m-%d%t%H:%M:%S');}
</Input>

<Input eventlog>
	Module	im_msvistalog
	<QueryXML>
		<QueryList>
        <Query Id="0">
            <Select Path="Application">*</Select>
			<Select Path="Security">*</Select>
			<Select Path="System">*</Select>
		</Query>
		</QueryList>
	</QueryXML>
</Input>

<Output gelf_sysmon>
    Module        om_tcp
    Host          192.168.0.10
	Port          12201
	Exec 		  $ShortMessage = $raw_event;
    OutputType    GELF_TCP
</Output>

<Output gelf_fwlog>
    Module        om_tcp
    Host          192.168.0.10
	Port          12202
	Exec 		  $ShortMessage = $raw_event;
    OutputType    GELF_TCP
</Output>

<Output gelf_windows>
    Module        om_tcp
    Host          192.168.0.10
	Port          12203
	Exec 		  $ShortMessage = $raw_event;
    OutputType    GELF_TCP
</Output>

<Route sysmon>
	Path          sysmon => gelf_sysmon
</Route>

<Route sysmon>
	Path          fwlog => gelf_fwlog
</Route>

<Route eventlog>
	Path          eventlog => gelf_windows
</Route>
```

