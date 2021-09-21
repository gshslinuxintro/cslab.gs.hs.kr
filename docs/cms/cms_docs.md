---
layout: page
title: Contest Management System
subtitle: 대회 관리 설명서
toc: true
#toc_title: Custom Title
menubar: example_menu
show_sidebar: false
---
<dl>

<html xmlns="http://www.w3.org/1999/xhtml" lang="en">
  <head>
    <meta http-equiv="X-UA-Compatible" content="IE=Edge" />
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title>CMS 1.5.dev0 documentation</title>
    <link rel="stylesheet" href="_static/alabaster.css" type="text/css" />
    <link rel="stylesheet" href="_static/pygments.css" type="text/css" />
    <link rel="stylesheet" type="text/css" href="_static/_static/css/badge_only.css" />
    <script type="text/javascript" id="documentation_options" data-url_root="./" src="_static/documentation_options.js"></script>
    <script type="text/javascript" src="_static/jquery.js"></script>
    <script type="text/javascript" src="_static/underscore.js"></script>
    <script type="text/javascript" src="_static/doctools.js"></script>
    <script type="text/javascript" src="_static/language_data.js"></script>
    <link rel="canonical" href="https://cms.readthedocs.io/en/v1.4/index.html" />
    <link rel="index" title="Index" href="genindex.html" />
    <link rel="search" title="Search" href="search.html" />
   
  <link rel="stylesheet" href="_static/custom.css" type="text/css" />
  
  
  <meta name="viewport" content="width=device-width, initial-scale=0.9, maximum-scale=0.9" />

  </head><body>
  

    <div class="document">
      <div class="documentwrapper">
        <div class="bodywrapper">
          

          <div class="body" role="main">
            
  <div class="section" id="welcome-to-cms-s-documentation">
<div class="toctree-wrapper compound">
<span id="document-Introduction"></span><div class="section" id="introduction">
<h1>Introduction<a class="headerlink" href="#introduction" title="Permalink to this headline">¶</a></h1>
<p>CMS (Contest Management System) is a software for organizing programming contests similar to well-known international contests like the IOI (International Olympiad in Informatics). It was written by and it received contributions from people involved in the organization of similar contests on a local, national and international level, and it is regularly used for such contests in many different countries. It is meant to be secure, extendable, adaptable to different situations and easy to use.</p>
<p>CMS is a complete, tested and well proved solution for managing a contest. However, it only provides limited tools for the development of the task data belonging to the contest (task statements, solutions, testcases, etc.). Also, the configuration of machines and network resources that host the contest is a responsibility of the contest administrators.</p>
<div class="section" id="general-structure">
<h2>General structure<a class="headerlink" href="#general-structure" title="Permalink to this headline">¶</a></h2>
<p>The system is organized in a modular way, with different services running (potentially) on different machines, and providing extendability via service replications on several machines.</p>
<p>The state of the contest is wholly kept on a PostgreSQL database (other DBMSs are not supported, as CMS relies on the Large Object (LO) feature of PostgreSQL). It is unlikely that in the future we will target different databases.</p>
<p>As long as the database is operating correctly, all other services can be started and stopped independently. For example, the administrator can quickly replace a broken machine with an identical one, which will take its roles (without having to move information from the broken machine). Of course, this also means that CMS is completely dependent on the database to run. In critical contexts, it is necessary to configure the database redundantly and be prepared to rapidly do a fail-over in case something bad happens. The choice of PostgreSQL as the database to use should ease this part, since there are many different, mature and well-known solutions to provide such redundance and fail-over procedures.</p>
</div>
<div class="section" id="services">
<h2>Services<a class="headerlink" href="#services" title="Permalink to this headline">¶</a></h2>
<p>CMS is composed of several services, that can be run on a single or on many servers. The core services are:</p>
<ul class="simple">
<li>LogService: collects all log messages in a single place;</li>
<li>ResourceService: collects data about the services running on the same server, and takes care of starting all of them with a single command;</li>
<li>Checker: simple heartbeat monitor for all services;</li>
<li>EvaluationService: organizes the queue of the submissions to compile or evaluate on the testcases, and dispatches these jobs to the workers;</li>
<li>Worker: actually runs the jobs in a sandboxed environment;</li>
<li>ScoringService: collects the outcomes of the submissions and computes the score;</li>
<li>ProxyService: sends the computed scores to the rankings;</li>
<li>PrintingService: processes files submitted for printing and sends them to a printer;</li>
<li>ContestWebServer: the webserver that the contestants will be interacting with;</li>
<li>AdminWebServer: the webserver to control and modify the parameters of the contests.</li>
</ul>
<p>Finally, RankingWebServer, whose duty is of course to show the ranking. This webserver is - on purpose - separated from the inner core of CMS in order to ease the creation of mirrors and restrict the number of people that can access services that are directly connected to the database.</p>
<p>Each of the core services is designed to be able to be killed and reactivated in a way that keeps the consistency of data, and does not block the functionalities provided by the other services.</p>
<p>Some of the services can be replicated on several machine: these are ResourceService (designed to be run on every machine), ContestWebServer and Worker.</p>
<p>In addition to services, CMS includes many command line tools. They help with importing, exporting and managing of contest data, and with testing.</p>
</div>
<div class="section" id="security-considerations">
<h2>Security considerations<a class="headerlink" href="#security-considerations" title="Permalink to this headline">¶</a></h2>
<p>With the exception of RWS, there are no cryptographic or authentication schemes between the various services or between the services and the database. Thus, it is mandatory to keep the services on a dedicated network, properly isolating it via firewalls from contestants or other people’s computers. This sort of operations, like also preventing contestants from communicating and cheating, is responsibility of the administrator and is not managed by CMS itself.</p>
</div>
</div>
<span id="document-Installation"></span><div class="section" id="installation">
<h1>Installation<a class="headerlink" href="#installation" title="Permalink to this headline">¶</a></h1>
<div class="section" id="dependencies-and-available-compilers">
<span id="installation-dependencies"></span><h2>Dependencies and available compilers<a class="headerlink" href="#dependencies-and-available-compilers" title="Permalink to this headline">¶</a></h2>
<p>These are our requirements (in particular we highlight those that are not usually installed by default) - previous versions may or may not work:</p>
<ul class="simple">
<li>build environment for the programming languages allowed in the competition;</li>
<li><a class="reference external" href="http://www.postgresql.org/">PostgreSQL</a> &gt;= 9.4;</li>
<li><a class="reference external" href="https://gcc.gnu.org/">GNU compiler collection</a> (in particular the C compiler <code class="docutils literal notranslate"><span class="pre">gcc</span></code>);</li>
<li><a class="reference external" href="http://www.python.org/">Python</a> &gt;= 3.6;</li>
<li><a class="reference external" href="http://libcg.sourceforge.net/">libcg</a>;</li>
<li><a class="reference external" href="https://www.tug.org/texlive/">TeX Live</a> (only for printing);</li>
<li><a class="reference external" href="https://www.gnu.org/software/a2ps/">a2ps</a> (only for printing).</li>
</ul>
<p>You will also require a Linux kernel with support for control groups and namespaces. Support has been in the Linux kernel since 2.6.32. Other distributions, or systems with custom kernels, may not have support enabled. At a minimum, you will need to enable the following Linux kernel options: <code class="docutils literal notranslate"><span class="pre">CONFIG_CGROUPS</span></code>, <code class="docutils literal notranslate"><span class="pre">CONFIG_CGROUP_CPUACCT</span></code>, <code class="docutils literal notranslate"><span class="pre">CONFIG_MEMCG</span></code> (previously called as <code class="docutils literal notranslate"><span class="pre">CONFIG_CGROUP_MEM_RES_CTLR</span></code>), <code class="docutils literal notranslate"><span class="pre">CONFIG_CPUSETS</span></code>, <code class="docutils literal notranslate"><span class="pre">CONFIG_PID_NS</span></code>, <code class="docutils literal notranslate"><span class="pre">CONFIG_IPC_NS</span></code>, <code class="docutils literal notranslate"><span class="pre">CONFIG_NET_NS</span></code>. It is anyway suggested to use Linux kernel version at least 3.8.</p>
<p>Then you require the compilation and execution environments for the languages you will use in your contest:</p>
<ul class="simple">
<li><a class="reference external" href="https://gcc.gnu.org/">GNU compiler collection</a> (for C and C++, respectively with executables <code class="docutils literal notranslate"><span class="pre">gcc</span></code> and <code class="docutils literal notranslate"><span class="pre">g++</span></code>);</li>
<li>for Java, your choice of a JDK, for example OpenJDK (but any other JDK behaving similarly is fine, for example Oracle’s);</li>
<li><a class="reference external" href="http://www.freepascal.org/">Free Pascal</a> (for Pascal, with executable <code class="docutils literal notranslate"><span class="pre">fpc</span></code>);</li>
<li><a class="reference external" href="http://www.python.org/">Python</a> &gt;= 2.7 (for Python, with executable <code class="docutils literal notranslate"><span class="pre">python2</span></code> or <code class="docutils literal notranslate"><span class="pre">python3</span></code>; in addition you will need <code class="docutils literal notranslate"><span class="pre">zip</span></code>);</li>
<li><a class="reference external" href="http://www.php.net">PHP</a> &gt;= 5 (for PHP, with executable <code class="docutils literal notranslate"><span class="pre">php</span></code>);</li>
<li><a class="reference external" href="https://www.haskell.org/ghc/">Glasgow Haskell Compiler</a> (for Haskell, with executable <code class="docutils literal notranslate"><span class="pre">ghc</span></code>);</li>
<li><a class="reference external" href="https://www.rust-lang.org/">Rust</a> (for Rust, with executable <code class="docutils literal notranslate"><span class="pre">rustc</span></code>);</li>
<li><a class="reference external" href="http://www.mono-project.com/docs/about-mono/languages/csharp/">C#</a> (for C#, with executable <code class="docutils literal notranslate"><span class="pre">mcs</span></code>).</li>
</ul>
<p>All dependencies can be installed automatically on most Linux distributions.</p>
<div class="section" id="ubuntu">
<h3>Ubuntu<a class="headerlink" href="#ubuntu" title="Permalink to this headline">¶</a></h3>
<p>On Ubuntu 20.04, one will need to run the following script to satisfy all dependencies:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span><span class="c1"># Feel free to change OpenJDK packages with your preferred JDK.</span>
sudo apt-get install build-essential openjdk-11-jdk-headless fp-compiler <span class="se">\</span>
    postgresql postgresql-client python3.8 cppreference-doc-en-html <span class="se">\</span>
    cgroup-lite libcap-dev zip

<span class="c1"># Only if you are going to use pip/venv to install python dependencies</span>
sudo apt-get install python3.8-dev libpq-dev libcups2-dev libyaml-dev <span class="se">\</span>
    libffi-dev python3-pip

<span class="c1"># Optional</span>
sudo apt-get install nginx-full python2.7 php7.4-cli php7.4-fpm <span class="se">\</span>
    phppgadmin texlive-latex-base a2ps haskell-platform rustc mono-mcs
</pre></div>
</div>
<p>The above commands provide a very essential Pascal environment. Consider installing the following packages for additional units: <cite>fp-units-base</cite>, <cite>fp-units-fcl</cite>, <cite>fp-units-misc</cite>, <cite>fp-units-math</cite> and <cite>fp-units-rtl</cite>.</p>
</div>
<div class="section" id="arch-linux">
<h3>Arch Linux<a class="headerlink" href="#arch-linux" title="Permalink to this headline">¶</a></h3>
<p>On Arch Linux, unofficial AUR packages can be found: <a class="reference external" href="http://aur.archlinux.org/packages/cms">cms</a> or <a class="reference external" href="http://aur.archlinux.org/packages/cms-git">cms-git</a>. However, if you do not want to use them, the following command will install almost all dependencies (some of them can be found in the AUR):</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>sudo pacman -S base-devel jdk8-openjdk fpc postgresql postgresql-client <span class="se">\</span>
    python libcap

<span class="c1"># Install the following from AUR.</span>
<span class="c1"># https://aur.archlinux.org/packages/libcgroup/</span>
<span class="c1"># https://aur.archlinux.org/packages/cppreference/</span>

<span class="c1"># Only if you are going to use pip/venv to install python dependencies</span>
sudo pacman -S --needed postgresql-libs libcups libyaml python-pip

<span class="c1"># Optional</span>
sudo pacman -S --needed nginx python2 php php-fpm phppgadmin texlive-core <span class="se">\</span>
    a2ps ghc rust mono
</pre></div>
</div>
</div>
</div>
<div class="section" id="preparation-steps">
<h2>Preparation steps<a class="headerlink" href="#preparation-steps" title="Permalink to this headline">¶</a></h2>
<p>Download <a class="reference external" href="https://github.com/cms-dev/cms/releases/download/v1.5.dev0/v1.5.dev0.tar.gz">CMS</a> 1.5.dev0 from GitHub as an archive, then extract it on your filesystem. You should then access the <code class="docutils literal notranslate"><span class="pre">cms</span></code> folder using a terminal.</p>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p>If you decided to <code class="docutils literal notranslate"><span class="pre">git</span> <span class="pre">clone</span></code> the repository instead of downloading the archive, and you didn’t use the <code class="docutils literal notranslate"><span class="pre">--recursive</span></code> option when cloning, then <strong>you need</strong> to issue the following command to fetch the source code of the sandbox:</p>
<div class="last highlight-bash notranslate"><div class="highlight"><pre><span></span>git submodule update --init
</pre></div>
</div>
</div>
<p>In order to run CMS there are some preparation steps to run (like installing the sandbox, compiling localization files, creating the <code class="docutils literal notranslate"><span class="pre">cmsuser</span></code>, and so on). You can either do all these steps by hand or you can run the following command:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>sudo python3 prerequisites.py install
</pre></div>
</div>
<p>This script will add you to the <code class="docutils literal notranslate"><span class="pre">cmsuser</span></code> group if you answer <code class="docutils literal notranslate"><span class="pre">Y</span></code> when asked. If you want to handle your groups by yourself, answer <code class="docutils literal notranslate"><span class="pre">N</span></code> and then run:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>sudo usermod -a -G cmsuser &lt;your user&gt;
</pre></div>
</div>
<p>You can verify to be in the group by issuing the command:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>groups
</pre></div>
</div>
<p>Remember to logout, to make the change effective.</p>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p class="last">Users in the group <code class="docutils literal notranslate"><span class="pre">cmsuser</span></code> will be able to launch the <code class="docutils literal notranslate"><span class="pre">isolate</span></code> program with root permission. They may exploit this to gain root privileges. It is then imperative that no untrusted user is allowed in the group <code class="docutils literal notranslate"><span class="pre">cmsuser</span></code>.</p>
</div>
</div>
<div class="section" id="installing-cms-and-its-python-dependencies">
<span id="installation-updatingcms"></span><h2>Installing CMS and its Python dependencies<a class="headerlink" href="#installing-cms-and-its-python-dependencies" title="Permalink to this headline">¶</a></h2>
<p>There are a number of ways to install CMS and its Python dependencies:</p>
<div class="section" id="method-1-global-installation-with-pip">
<h3>Method 1: Global installation with pip<a class="headerlink" href="#method-1-global-installation-with-pip" title="Permalink to this headline">¶</a></h3>
<p>There are good reasons to install CMS and its Python dependencies via pip (Python Package Index) instead of your package manager (e.g. apt-get). For example: two different Linux distro (or two different versions of the same distro) may offer two different versions of <code class="docutils literal notranslate"><span class="pre">python-sqlalchemy</span></code>. When using pip, you can choose to install a <em>specific version</em> of <code class="docutils literal notranslate"><span class="pre">sqlalchemy</span></code> that is known to work correctly with CMS.</p>
<p>Assuming you have <code class="docutils literal notranslate"><span class="pre">pip</span></code> installed, you can do this:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>sudo pip3 install -r requirements.txt
sudo python3 setup.py install
</pre></div>
</div>
<p>This command installs python dependencies globally. Note that on some distros, like Arch Linux, this might interfere with the system package manager. If you want to perform the installation in your home folder instead, then you can do this instead:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>pip3 install --user -r requirements.txt
python3 setup.py install --user
</pre></div>
</div>
</div>
<div class="section" id="method-2-virtual-environment">
<h3>Method 2: Virtual environment<a class="headerlink" href="#method-2-virtual-environment" title="Permalink to this headline">¶</a></h3>
<p>An alternative method to perform the installation is with a <a class="reference external" href="https://virtualenv.pypa.io/en/latest/">virtual environment</a>, which is an isolated Python environment that you can put wherever you like and that can be activated/deactivated at will.</p>
<p>You will need to create a virtual environment somewhere in your filesystem. For example, let’s assume that you decided to create it under your home directory (as <code class="docutils literal notranslate"><span class="pre">~/cms_venv</span></code>):</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>python3 -m venv ~/cms_venv
</pre></div>
</div>
<p>To activate it:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span><span class="nb">source</span> ~/cms_venv/bin/activate
</pre></div>
</div>
<p>After the activation, the <code class="docutils literal notranslate"><span class="pre">pip</span></code> command will <em>always</em> be available (even if it was not available globally, e.g. because you did not install it). In general, every python command (python, pip) will refer to their corresponding virtual version. So, you can install python dependencies by issuing:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>pip3 install -r requirements.txt
python3 setup.py install
</pre></div>
</div>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p>Once you finished using CMS, you can deactivate the virtual environment by issuing:</p>
<div class="last highlight-bash notranslate"><div class="highlight"><pre><span></span>deactivate
</pre></div>
</div>
</div>
</div>
<div class="section" id="method-3-using-apt-get-on-ubuntu">
<h3>Method 3: Using <code class="docutils literal notranslate"><span class="pre">apt-get</span></code> on Ubuntu<a class="headerlink" href="#method-3-using-apt-get-on-ubuntu" title="Permalink to this headline">¶</a></h3>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p class="last">It is usually possible to install python dependencies using your Linux distribution’s package manager. However, keep in mind that the version of each package is controlled by the package mantainers and could be too new or too old for CMS. <strong>On Ubuntu, this is generally not the case</strong> since we try to build on the python packages that are available for the current LTS version.</p>
</div>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p class="last">On Ubuntu 20.04, the shipped version of <code class="docutils literal notranslate"><span class="pre">python3-gevent</span></code> is too old to support the system Python 3 version. After installing other packages from the repositories, you should still install <code class="docutils literal notranslate"><span class="pre">gevent&gt;=1.5,&lt;1.6</span></code>, for example, using the <code class="docutils literal notranslate"><span class="pre">pip</span></code> method above.</p>
</div>
<p>To install CMS and its Python dependencies on Ubuntu, you can issue:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>sudo python3 setup.py install

sudo apt-get install python3-setuptools python3-tornado4 python3-psycopg2 <span class="se">\</span>
     python3-sqlalchemy python3-psutil python3-netifaces python3-pycryptodome <span class="se">\</span>
     python3-bs4 python3-coverage python3-requests python3-werkzeug <span class="se">\</span>
     python3-gevent python3-bcrypt python3-chardet patool python3-babel <span class="se">\</span>
     python3-xdg python3-jinja2

<span class="c1"># Optional.</span>
<span class="c1"># sudo apt-get install python3-yaml python3-sphinx python3-cups python3-pypdf2</span>
</pre></div>
</div>
</div>
<div class="section" id="method-4-using-pacman-on-arch-linux">
<h3>Method 4: Using <code class="docutils literal notranslate"><span class="pre">pacman</span></code> on Arch Linux<a class="headerlink" href="#method-4-using-pacman-on-arch-linux" title="Permalink to this headline">¶</a></h3>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p class="last">It is usually possible to install python dependencies using your Linux distribution’s package manager. However, keep in mind that the version of each package is controlled by the package mantainers and could be too new or too old for CMS. <strong>This is especially true for Arch Linux</strong>, which is a bleeding edge distribution.</p>
</div>
<p>To install CMS python dependencies on Arch Linux (again: assuming you did not use the aforementioned AUR packages), you can issue:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>sudo python3 setup.py install

sudo pacman -S --needed python-setuptools python-tornado python-psycopg2 <span class="se">\</span>
     python-sqlalchemy python-psutil python-netifaces python-pycryptodome <span class="se">\</span>
     python-beautifulsoup4 python-coverage python-requests python-werkzeug <span class="se">\</span>
     python-gevent python-bcrypt python-chardet python-babel python-xdg <span class="se">\</span>
     python-jinja

<span class="c1"># Install the following from AUR.</span>
<span class="c1"># https://aur.archlinux.org/packages/patool/</span>

<span class="c1"># Optional.</span>
<span class="c1"># sudo pacman -S --needed python-yaml python-sphinx python-pycups</span>
<span class="c1"># Optionally install the following from AUR.</span>
<span class="c1"># https://aur.archlinux.org/packages/python-pypdf2/</span>
</pre></div>
</div>
</div>
</div>
<div class="section" id="configuring-the-worker-machines">
<h2>Configuring the worker machines<a class="headerlink" href="#configuring-the-worker-machines" title="Permalink to this headline">¶</a></h2>
<p>Worker machines need to be carefully set up in order to ensure that evaluation results are valid and consistent. Just running the evaluations under isolate does not achieve this: for example, if the machine has an active swap partition, memory limit will not be honored.</p>
<p>Apart from validity, there are many possible tweaks to reduce the variability in resource usage of an evaluation.</p>
<p>We suggest following isolate’s <a class="reference external" href="https://github.com/ioi/isolate/blob/c679ae936d8e8d64e5dab553bdf1b22261324315/isolate.1.txt#L292">guidelines</a> for reproducible results.</p>
</div>
<div class="section" id="running-cms-non-installed">
<span id="installation-running-cms-non-installed"></span><h2>Running CMS non-installed<a class="headerlink" href="#running-cms-non-installed" title="Permalink to this headline">¶</a></h2>
<p>To run CMS without installing it in the system, you need first to build the prerequisites:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>python3 prerequisites.py build
</pre></div>
</div>
<p>There are still a few steps to complete manually in this case. First, add CMS and isolate to the path and create the configuration files:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span><span class="nb">export</span> <span class="nv">PATH</span><span class="o">=</span><span class="nv">$PATH</span>:./isolate/
<span class="nb">export</span> <span class="nv">PYTHONPATH</span><span class="o">=</span>./
cp config/cms.conf.sample config/cms.conf
cp config/cms.ranking.conf.sample config/cms.ranking.conf
</pre></div>
</div>
<p>Second, perform these tasks (that require root permissions):</p>
<ul class="simple">
<li>create the <code class="docutils literal notranslate"><span class="pre">cmsuser</span></code> user and a group with the same name;</li>
<li>add your user to the <code class="docutils literal notranslate"><span class="pre">cmsuser</span></code> group;</li>
<li>set isolate to be owned by root:cmsuser, and set its suid bit.</li>
</ul>
<p>For example:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>sudo useradd cmsuser
sudo usermod -a -G cmsuser &lt;your user&gt;
sudo chown root:cmsuser ./isolate/isolate
sudo chmod u+s ./isolate/isolate
</pre></div>
</div>
</div>
<div class="section" id="updating-cms">
<h2>Updating CMS<a class="headerlink" href="#updating-cms" title="Permalink to this headline">¶</a></h2>
<p>As CMS develops, the database schema it uses to represent its data may be updated and new versions may introduce changes that are incompatible with older versions.</p>
<p>To preserve the data stored on the database you need to dump it on the filesystem using <code class="docutils literal notranslate"><span class="pre">cmsDumpExporter</span></code> <strong>before you update CMS</strong> (i.e. with the old version).</p>
<p>You can then update CMS and reset the database schema by running:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>cmsDropDB
cmsInitDB
</pre></div>
</div>
<p>To load the previous data back into the database you can use <code class="docutils literal notranslate"><span class="pre">cmsDumpImporter</span></code>: it will adapt the data model automatically on-the-fly (you can use <code class="docutils literal notranslate"><span class="pre">cmsDumpUpdater</span></code> to store the updated version back on disk and speed up future imports).</p>
</div>
</div>
<span id="document-Running CMS"></span><div class="section" id="running-cms">
<h1>Running CMS<a class="headerlink" href="#running-cms" title="Permalink to this headline">¶</a></h1>
<div class="section" id="configuring-the-db">
<h2>Configuring the DB<a class="headerlink" href="#configuring-the-db" title="Permalink to this headline">¶</a></h2>
<p>The first thing to do is to create the user and the database. If you’re on Ubuntu, you need to login as the postgres user first:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>sudo su - postgres
</pre></div>
</div>
<p>Then, to create the user (which does not need to be a superuser, nor be able to create databases nor roles) and the database, you need the following commands:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>createuser --username<span class="o">=</span>postgres --pwprompt cmsuser
createdb --username<span class="o">=</span>postgres --owner<span class="o">=</span>cmsuser cmsdb
psql --username<span class="o">=</span>postgres --dbname<span class="o">=</span>cmsdb --command<span class="o">=</span><span class="s1">&#39;ALTER SCHEMA public OWNER TO cmsuser&#39;</span>
psql --username<span class="o">=</span>postgres --dbname<span class="o">=</span>cmsdb --command<span class="o">=</span><span class="s1">&#39;GRANT SELECT ON pg_largeobject TO cmsuser&#39;</span>
</pre></div>
</div>
<p>The last two lines are required to give the PostgreSQL user some privileges which it does not have by default, despite being the database owner.</p>
<p>Then you may need to adjust the CMS configuration to contain the correct database parameters. See <a class="reference internal" href="#running-cms-configuring-cms"><span class="std std-ref">Configuring CMS</span></a>.</p>
<p>Finally you have to create the database schema for CMS, by running:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>cmsInitDB
</pre></div>
</div>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p>If you are going to use CMS services on different hosts from the one where PostgreSQL is running, you also need to instruct it to accept the connections from the services. To do so, you need to change the listening address of PostgreSQL in <code class="file docutils literal notranslate"><span class="pre">postgresql.conf</span></code>, for example like this:</p>
<div class="highlight-default notranslate"><div class="highlight"><pre><span></span><span class="n">listen_addresses</span> <span class="o">=</span> <span class="s1">&#39;127.0.0.1,192.168.0.x&#39;</span>
</pre></div>
</div>
<p>Moreover, you need to change the HBA (a sort of access control list for PostgreSQL) to accept login requests from outside localhost. Open the file <code class="file docutils literal notranslate"><span class="pre">pg_hba.conf</span></code> and add a line like this one:</p>
<div class="last highlight-default notranslate"><div class="highlight"><pre><span></span><span class="n">host</span>  <span class="n">cmsdb</span>  <span class="n">cmsuser</span>  <span class="mf">192.168.0.0</span><span class="o">/</span><span class="mi">24</span>  <span class="n">md5</span>
</pre></div>
</div>
</div>
</div>
<div class="section" id="configuring-cms">
<span id="running-cms-configuring-cms"></span><h2>Configuring CMS<a class="headerlink" href="#configuring-cms" title="Permalink to this headline">¶</a></h2>
<p>There are two configuration files, one for CMS itself and one for the rankings. Samples for both files are in the directory <a class="reference external" href="https://github.com/cms-dev/cms/tree/v1.5.dev0/config/">config/</a>. You want to copy them to the same file names but without the <code class="docutils literal notranslate"><span class="pre">.sample</span></code> suffix (that is, to <code class="file docutils literal notranslate"><span class="pre">config/cms.conf</span></code> and <code class="file docutils literal notranslate"><span class="pre">config/cms.ranking.conf</span></code>) before modifying them.</p>
<ul class="simple">
<li><code class="file docutils literal notranslate"><span class="pre">cms.conf</span></code> is intended to be the same on all machines; all configurations options are explained in the file; of particular importance is the definition of <code class="docutils literal notranslate"><span class="pre">core_services</span></code>, that specifies where and how many services are going to be run, and the connecting line for the database, in which you need to specify the name of the user created above and its password.</li>
<li><code class="file docutils literal notranslate"><span class="pre">cms.ranking.conf</span></code> is not necessarily meant to be the same on each server that will host a ranking, since it just controls settings relevant for one single server. The addresses and log-in information of each ranking must be the same as the ones found in <code class="file docutils literal notranslate"><span class="pre">cms.conf</span></code>.</li>
</ul>
<p>These files are a pretty good starting point if you want to try CMS. There are some mandatory changes to do though:</p>
<ul class="simple">
<li>you must change the connection string given in <code class="docutils literal notranslate"><span class="pre">database</span></code>; this usually means to change username, password and database with the ones you chose before;</li>
<li>if you are running low on disk space, you may want to make sure <code class="docutils literal notranslate"><span class="pre">keep_sandbox</span></code> is set to <code class="docutils literal notranslate"><span class="pre">false</span></code>;</li>
</ul>
<p>If you are organizing a real contest, you must also change <code class="docutils literal notranslate"><span class="pre">secret_key</span></code> to a random key (the admin interface will suggest one if you visit it when <code class="docutils literal notranslate"><span class="pre">secret_key</span></code> is the default). You will also need to think about how to distribute your services and change <code class="docutils literal notranslate"><span class="pre">core_services</span></code> accordingly. Finally, you should change the ranking section of <code class="file docutils literal notranslate"><span class="pre">cms.conf</span></code>, and <code class="file docutils literal notranslate"><span class="pre">cms.ranking.conf</span></code>, using non-trivial username and password.</p>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p class="last">As the name implies, the value of <code class="docutils literal notranslate"><span class="pre">secret_key</span></code> must be kept confidential. If a contestant knows it (for example because you are using the default value), they may be easily able to log in as another contestant.</p>
</div>
<p>The configuration files get copied automatically by the <code class="docutils literal notranslate"><span class="pre">prerequisites.py</span></code> script, so you can either run <code class="docutils literal notranslate"><span class="pre">sudo</span> <span class="pre">./prerequisites.py</span> <span class="pre">install</span></code> again (answering “Y” when questioned about overwriting old configuration files) or you could simply edit the previously installed configuration files (which are usually found in <code class="docutils literal notranslate"><span class="pre">/usr/local/etc/</span></code> or <code class="docutils literal notranslate"><span class="pre">/etc/</span></code>), if you do not plan on running that command ever again.</p>
</div>
<div class="section" id="id1">
<h2>Running CMS<a class="headerlink" href="#id1" title="Permalink to this headline">¶</a></h2>
<p>Here we will assume you installed CMS. If not, you should replace all commands path with the appropriate local versions (for example, <code class="docutils literal notranslate"><span class="pre">cmsLogService</span></code> becomes <a class="reference external" href="https://github.com/cms-dev/cms/blob/v1.5.dev0/./scripts/cmsLogService">./scripts/cmsLogService</a>).</p>
<p>At this point, you should have CMS installed on all the machines you want run services on, with the same configuration file, and a running PostgreSQL instance. To run CMS, you need a contest in the database. To create a contest, follow <a class="reference internal" href="index.html#document-Creating a contest"><span class="doc">these instructions</span></a>.</p>
<p>CMS is composed of a number of services, potentially replicated several times, and running on several machines. You can start all the services by hand, but this is a tedious task. Luckily, there is a service (ResourceService) that takes care of starting all the services on the machine it is running, limiting thus the number of binaries you have to run. Services started by ResourceService do not show their logs to the standard output; so it is expected that you run LogService to inspect the logs as they arrive (logs are also saved to disk). To start LogService, you need to issue, in the machine specified in cms.conf for LogService, this command:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>cmsLogService <span class="m">0</span>
</pre></div>
</div>
<p>where <code class="docutils literal notranslate"><span class="pre">0</span></code> is the “shard” of LogService you want to run. Since there must be only one instance of LogService, it is safe to let CMS infer that the shard you want is the 0-th, and so an equivalent command is</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>cmsLogService
</pre></div>
</div>
<p>After LogService is running, you can start ResourceService on each machine involved, instructing it to load all the other services:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>cmsResourceService -a
</pre></div>
</div>
<p>The flag <code class="docutils literal notranslate"><span class="pre">-a</span></code> informs ResourceService that it has to start all other services, and we have omitted again the shard number since, even if ResourceService is replicated, there must be only one of it in each machine. If you have a funny network configuration that confuses CMS, just give explicitly the shard number. In any case, ResourceService will ask you the contest to load, and will start all the other services. You should start see logs flowing in the LogService terminal.</p>
<p>Note that it is your duty to keep CMS’s configuration synchronized among the machines.</p>
<p>You should now be able to start exploring the admin interface, by default at <a class="reference external" href="http://localhost:8889/">http://localhost:8889/</a>. The interface is accessible with an admin account, which you need to create first using the AddAdmin command, for example:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>cmsAddAdmin name
</pre></div>
</div>
<p>CMS will create an admin account with username “name” and a random password that will be printed by the command. You can log in with this credentials, and then use the admin interface to modify the account or add other accounts.</p>
</div>
<div class="section" id="recommended-setup">
<span id="running-cms-recommended-setup"></span><h2>Recommended setup<a class="headerlink" href="#recommended-setup" title="Permalink to this headline">¶</a></h2>
<p>Of course, the number of servers one needs to run a contest depends on many factors (number of participants, length of the contest, economical issues, more technical matters…). We recommend that, for fairness, each Worker runs an a dedicated machine (i.e., without other CMS services beyond ResourceService).</p>
<p>As for the distribution of services, usually there is one ResourceService for each machine, one instance for each of LogService, ScoringService, Checker, EvaluationService, AdminWebServer, and one or more instances of ContestWebServer and Worker. Again, if there are more than one Worker, we recommend to run them on different machines.</p>
<p>The developers of isolate (the sandbox CMS uses) provide a script, <code class="file docutils literal notranslate"><span class="pre">isolate-check-environment</span></code> that verifies your system is able to produce evaluations as fair and reproducible as possible. We recommend to run it and follow its suggestions on all machines where a Worker is running. You can download it <a class="reference external" href="https://github.com/ioi/isolate/blob/master/isolate-check-environment">here</a>.</p>
<p>We suggest using CMS over Ubuntu. Yet, CMS can be successfully run on different Linux distributions. Non-Linux operating systems are not supported.</p>
<p>We recommend using nginx in front of the (one or more) <code class="file docutils literal notranslate"><span class="pre">cmsContestWebServer</span></code> instances serving the contestant interface. Using a load balancer is required when having multiple instances of <code class="file docutils literal notranslate"><span class="pre">cmsContestWebServer</span></code>, but even in case of a single instance, we suggest using nginx to secure the connection, providing an HTTPS endpoint and redirecting it to <code class="file docutils literal notranslate"><span class="pre">cmsContestWebServer</span></code>’s HTTP interface.</p>
<p>See <a class="reference external" href="https://github.com/cms-dev/cms/blob/v1.5.dev0/config/nginx.conf.sample">config/nginx.conf.sample</a> for a sample nginx configuration. This file probably needs to be adapted to your distribution if it is not Ubuntu: try to merge it with the file you find installed by default. For additional information see the official nginx <a class="reference external" href="http://wiki.nginx.org/HttpUpstreamModule">documentation</a> and <a class="reference external" href="http://wiki.nginx.org/LoadBalanceExample">examples</a>. Note that without the <code class="docutils literal notranslate"><span class="pre">ip_hash</span></code> option some CMS features might not always work as expected.</p>
</div>
<div class="section" id="logs">
<h2>Logs<a class="headerlink" href="#logs" title="Permalink to this headline">¶</a></h2>
<p>When the services are running, log messages are streamed to the log
service. This is the meaning of the log levels:</p>
<ul class="simple">
<li>debug: they are just for development; in the default configuration, they are not printed;</li>
<li>info: they inform you on what is going on in the system and that everything is fine;</li>
<li>warning: something went wrong or was slightly unexpected, but CMS knew how to handle it, or someone fed inappropriate data to CMS (by error or on purpose); you may want to check these as they may evolve into errors or unexpected behaviors, or hint that a contestant is trying to cheat;</li>
<li>error: an unexpected condition that should not have happened; you are encouraged to take actions to fix them, but the service will continue to work (most of the time, ignoring the error and the data connected to it);</li>
<li>critical: a condition so unexpected that the service is really startled and refuses to continue working; you are forced to take action because with high probability the service will continue having the same problem upon restarting.</li>
</ul>
<p>Warning, error, and critical log messages are also displayed in the main page of AdminWebServer.</p>
</div>
</div>
<span id="document-Data model"></span><div class="section" id="data-model">
<h1>Data model<a class="headerlink" href="#data-model" title="Permalink to this headline">¶</a></h1>
<p>CMS is used in many different settings: single onsite contests, with remote participations, short training camps, always-on training websites, university assignments… Its data model needs to support all of these, and therefore might be a bit surprising if you only think of the first use case.</p>
<p>In the following, we explain the main objects in CMS’s data model.</p>
<div class="section" id="users">
<h2>Users<a class="headerlink" href="#users" title="Permalink to this headline">¶</a></h2>
<p>Users are the accounts for contestants; a user represents a person, which can participate to zero, one or many contests.</p>
</div>
<div class="section" id="participations">
<h2>Participations<a class="headerlink" href="#participations" title="Permalink to this headline">¶</a></h2>
<p>Participations contain the interactions of users and a contest. In particular all of the following are associated to a participation: the submissions sent, their results, questions asked, communications from contest admins.</p>
</div>
<div class="section" id="contests">
<h2>Contests<a class="headerlink" href="#contests" title="Permalink to this headline">¶</a></h2>
<p>A contest is a collection of tasks to be solved, and participations of users trying to solve them.</p>
<p>It is <a class="reference internal" href="index.html#document-Configuring a contest"><span class="doc">very configurable</span></a> with respect of timing (start and end times, how much time each contestant has), logistic (how contestants login), permissions (what contestants can or cannot do, and how often), scoring, and more.</p>
<p>They are mainly thought as limited in duration to a few hours, but this is not a requirement: contests can be very long running.</p>
</div>
<div class="section" id="tasks-and-datasets">
<h2>Tasks and datasets<a class="headerlink" href="#tasks-and-datasets" title="Permalink to this headline">¶</a></h2>
<p>A task is one of the problems to solve within a contest. A task cannot be associated to more than one contest, but you can have tasks temporarily not associated to any.</p>
<p>Tasks store additional configurations that might override or alter the configurations at the contest level.</p>
<p>A task can have one or more datasets. The complete task data is shared between these two objects: task contains more contest-visible data, such as name, statement, constraints on the submissions; datasets instead contain instruction on how to compile, evaluate and score the submissions. The information in the task object should not change, as they are visible to the contestants and influence their interaction during the contest, whereas those in the dataset object could change without the contestants noticing.</p>
<p>The dataset used to display information to contestants is said to be active. For example, a second, inactive dataset can be created to fix an incorrect testcase; evaluation could progress in parallel until the inactive dataset is deemed correct, and eventually the switch to make it active could happen at once.</p>
<p>See the section on <a class="reference internal" href="index.html#document-Task versioning"><span class="doc">task versioning</span></a> for more details on how to use datasets.</p>
</div>
<div class="section" id="submissions-results-and-tokens">
<h2>Submissions, results and tokens<a class="headerlink" href="#submissions-results-and-tokens" title="Permalink to this headline">¶</a></h2>
<p>Submissions are associated to a participation and a task. In addition to these, the judging of a submission depends on the dataset used in the compilation, evaluation and scoring phases. Therefore, a submission result is associated also to a specific dataset.</p>
<p>Similarly to submissions and submission results, we have user tests and user tests results. User tests are a way to allow contestants to run their source on the sandboxed environment, for example to diagnose errors or test timings.</p>
<p>Tokens are requests from a contestant to see additional details about the result of a submission they are associated to the submission (not to the submission result, so if the admins change dataset, the contestant can still see the detailed results).</p>
</div>
<div class="section" id="announcements-questions-and-messages">
<h2>Announcements, questions and messages<a class="headerlink" href="#announcements-questions-and-messages" title="Permalink to this headline">¶</a></h2>
<p>Announcements are contest-level communications sent by the admins and visible to all contestants.</p>
<p>Questions and messages instead are communication between a specific contestant and the admins. The difference is that questions are initiated by the contestants and expect an answer from the admins, whereas messages are initiated by the admins and contestants cannot reply.</p>
</div>
<div class="section" id="admins">
<h2>Admins<a class="headerlink" href="#admins" title="Permalink to this headline">¶</a></h2>
<p>CMS stores administrative accounts to use AdminWebServer. Accounts can be useful to offer different permissions to different sets of people. There are three levels of permissions: read-only, full permissions, and communication only (that is, these admins can only answer questions, send announcements and messages).</p>
</div>
</div>
<span id="document-Creating a contest"></span><div class="section" id="creating-a-contest">
<h1>Creating a contest<a class="headerlink" href="#creating-a-contest" title="Permalink to this headline">¶</a></h1>
<div class="section" id="creating-a-contest-from-scratch">
<h2>Creating a contest from scratch<a class="headerlink" href="#creating-a-contest-from-scratch" title="Permalink to this headline">¶</a></h2>
<p>The most immediate (but often less practical) way to create a contest in CMS is using the admin interface. You can start the AdminWebServer using the command <code class="docutils literal notranslate"><span class="pre">cmsAdminWebServer</span></code> (or using the ResourceService).</p>
<p>After that, you can connect to the server using the address and port specified in <code class="file docutils literal notranslate"><span class="pre">cms.conf</span></code>; by default, <a class="reference external" href="http://localhost:8889/">http://localhost:8889/</a>. Here, you can create a contest clicking on the link in the left column. After this, you must similarly add tasks and users.</p>
<p>Since the details of contests, tasks and users usually live somewhere in the filesystem, it is more practical to use an importer to create them in CMS automatically.</p>
</div>
<div class="section" id="creating-a-contest-from-the-filesystem">
<h2>Creating a contest from the filesystem<a class="headerlink" href="#creating-a-contest-from-the-filesystem" title="Permalink to this headline">¶</a></h2>
<p>CMS philosophy is that, unless you want, it should not change how you develop tasks, or how you store contests, tasks and user information.</p>
<p>To achieve this goal, CMS has tools to import a contest from a custom filesystem description. There are commands which read a filesystem description and use it to create contests, tasks, or users. Specifically: the <code class="docutils literal notranslate"><span class="pre">cmsImportContest</span></code>, <code class="docutils literal notranslate"><span class="pre">cmsImportTasl</span></code>, <code class="docutils literal notranslate"><span class="pre">cmsImportUser</span></code> commands (by default) will analyze the directory given as first argument and detect if it can be loaded (respectively) as a new contest, task, user. Run the commands with a <code class="docutils literal notranslate"><span class="pre">-h</span></code> or <code class="docutils literal notranslate"><span class="pre">--help</span></code> flag in order to better understand how they can be used.</p>
<p>In order to make these tools compatible with your filesystem format, you have to write a Python module that converts your filesystem description to the internal CMS representation of the contest. You have to extend the classes <code class="docutils literal notranslate"><span class="pre">ContestLoader</span></code>, <code class="docutils literal notranslate"><span class="pre">TaskLoader</span></code>, <code class="docutils literal notranslate"><span class="pre">UserLoader</span></code> defined in <a class="reference external" href="https://github.com/cms-dev/cms/blob/v1.5.dev0/cmscontrib/loaders/base_loader.py">cmscontrib/loaders/base_loader.py</a>, implementing missing methods as required by the docstrings (or use one of the existing loaders in <code class="file docutils literal notranslate"><span class="pre">cmscontrib/loaders/</span></code> as a template). If you do not use complex task types, or many different configurations, loaders can be very simple.</p>
<p>Out of the box, CMS offers loaders for two formats:</p>
<ul class="simple">
<li>The Italian filesystem format supports all the features of CMS. No compatibility in time is guaranteed with this format. If you want to use it, an example of a contest written in this format is in <a class="reference external" href="https://github.com/cms-dev/con_test">this GitHub repository</a>, while its explanation is <a class="reference internal" href="index.html#document-External contest formats"><span class="doc">here</span></a>.</li>
<li>The <a class="reference external" href="https://polygon.codeforces.com/">Polygon format</a>, which is the format used in several contests and by Codeforces. Polygon does not support all of CMS features, but having this importer is especially useful if you have a big repository of tasks in this format.</li>
</ul>
<p>CMS also has several convenience scripts to add data to the database specifying it on the command line, or to remove data from the database. Look in <code class="file docutils literal notranslate"><span class="pre">cmscontrib</span></code> or at commands starting with <code class="docutils literal notranslate"><span class="pre">cmsAdd</span></code> or <code class="docutils literal notranslate"><span class="pre">cmsRemove</span></code>.</p>
</div>
<div class="section" id="creating-a-contest-from-an-exported-contest">
<h2>Creating a contest from an exported contest<a class="headerlink" href="#creating-a-contest-from-an-exported-contest" title="Permalink to this headline">¶</a></h2>
<p>This option is not really suited for creating new contests but to store and move contest already used in CMS. If you have the dump of a contest exported from CMS, you can import it with <code class="docutils literal notranslate"><span class="pre">cmsDumpImporter</span> <span class="pre">&lt;source&gt;</span></code>, where <code class="docutils literal notranslate"><span class="pre">&lt;source&gt;</span></code> is the archive filename or directory of the contest.</p>
</div>
</div>
<span id="document-Configuring a contest"></span><div class="section" id="configuring-a-contest">
<h1>Configuring a contest<a class="headerlink" href="#configuring-a-contest" title="Permalink to this headline">¶</a></h1>
<p>In the following text “user” and “contestant” are used interchangeably. A “participation” is an instance of a user participating in a specific contest. See <a class="reference internal" href="index.html#document-Data model"><span class="doc">here</span></a> for more details.</p>
<p>Configuration parameters will be referred to using their internal name, but it should always be easy to infer what fields control them in the AWS interface by using their label.</p>
<div class="section" id="limitations">
<span id="configuringacontest-limitations"></span><h2>Limitations<a class="headerlink" href="#limitations" title="Permalink to this headline">¶</a></h2>
<p>Contest administrators can limit the ability of users to submit submissions and user_tests, by setting the following parameters:</p>
<ul>
<li><p class="first"><code class="docutils literal notranslate"><span class="pre">max_submission_number</span></code> / <code class="docutils literal notranslate"><span class="pre">max_user_test_number</span></code></p>
<p>These set, respectively, the maximum number of submissions or user tests that will be accepted for a certain user. Any attempt to send in additional submissions or user tests after that limit has been reached will fail.</p>
</li>
<li><p class="first"><code class="docutils literal notranslate"><span class="pre">min_submission_interval</span></code> / <code class="docutils literal notranslate"><span class="pre">min_user_test_interval</span></code></p>
<p>These set, respectively, the minimum amount of time, in seconds, the user is required to wait after a submission or user test has been submitted before they are allowed to send in new ones. Any attempt to submit a submission or user test before this timeout has expired will fail.</p>
</li>
</ul>
<p>The limits can be set both for individual tasks and for the whole contest. A submission or user test is accepted if it verifies the conditions on both the task <em>and</em> the contest. This means that a submission or user test will be accepted if the number of submissions or user tests received so far for that task is strictly less that the task’s maximum number <em>and</em> the number of submissions or user tests received so far for the whole contest (i.e. in all tasks) is strictly less than the contest’s maximum number. The same holds for the minimum interval too: a submission or user test will be accepted if the time passed since the last submission or user test for that task is greater than the task’s minimum interval <em>and</em> the time passed since the last submission or user test received for the whole contest (i.e. in any of the tasks) is greater than the contest’s minimum interval.</p>
<p>Each of these fields can be left unset to prevent the corresponding limitation from being enforced.</p>
</div>
<div class="section" id="feedback-to-contestants">
<h2>Feedback to contestants<a class="headerlink" href="#feedback-to-contestants" title="Permalink to this headline">¶</a></h2>
<p>Each testcase can be marked as public or private. After sending a submission, a contestant can always see its results on the public testcases: a brief passed / partial / not passed status for each testcase, and the partial score that is computable from the public testcases only. Note that input and output data are always hidden.</p>
<p>Tokens were introduced to provide contestants with limited access to the detailed results of their submissions on the private testcases as well. If a contestant uses a token on a submission, then they will be able to see its result on all testcases, and the global score.</p>
<div class="section" id="tokens-rules">
<span id="configuringacontest-tokens"></span><h3>Tokens rules<a class="headerlink" href="#tokens-rules" title="Permalink to this headline">¶</a></h3>
<p>Each contestant have a set of available tokens at their disposal; when they use a token it is taken from this set, and cannot be use again. These sets are managed by CMS according to rules defined by the contest administrators, as explained later in this section.</p>
<p>There are two types of tokens: contest-tokens and task-tokens. When a contestant uses a token to unlock a submission, they are really using one token of each type, and therefore needs to have both available. As the names suggest, contest-tokens are bound to the contest while task-tokens are bound to a specific task. That means that there is just one set of contest-tokens but there can be many sets of task-tokens (precisely one for every task). These sets are controlled independently by rules defined either on the contest or on the task.</p>
<p>A token set can be disabled (i.e. there will never be tokens available for use), infinite (i.e. there will always be tokens available for use) or finite. This setting is controlled by the <code class="docutils literal notranslate"><span class="pre">token_mode</span></code> parameter.</p>
<p>If the token set is finite it can be effectively represented by a non-negative integer counter: its cardinality. When the contest starts (or when the user starts its per-user time-frame, see <a class="reference internal" href="#configuringacontest-usaco-like-contests"><span class="std std-ref">USACO-like contests</span></a>) the set will be filled with <code class="docutils literal notranslate"><span class="pre">token_gen_initial</span></code> tokens (i.e. the counter is set to <code class="docutils literal notranslate"><span class="pre">token_gen_initial</span></code>). If the set is not empty (i.e. the counter is not zero) the user can use a token. After that, the token is discarded (i.e. the counter is decremented by one). New tokens can be generated during the contest: <code class="docutils literal notranslate"><span class="pre">token_gen_number</span></code> new tokens will be given to the user after every <code class="docutils literal notranslate"><span class="pre">token_gen_interval</span></code> minutes from the start (note that <code class="docutils literal notranslate"><span class="pre">token_gen_number</span></code> can be zero, thus disabling token generation). If <code class="docutils literal notranslate"><span class="pre">token_gen_max</span></code> is set, the set cannot contain more than <code class="docutils literal notranslate"><span class="pre">token_gen_max</span></code> tokens (i.e. the counter is capped at that value). Generation will continue but will be ineffective until the contestant uses a token. Unset <code class="docutils literal notranslate"><span class="pre">token_gen_max</span></code> to disable this limit.</p>
<p>The use of tokens can be limited with <code class="docutils literal notranslate"><span class="pre">token_max_number</span></code> and <code class="docutils literal notranslate"><span class="pre">token_min_interval</span></code>: users cannot use more that <code class="docutils literal notranslate"><span class="pre">token_max_number</span></code> tokens in total (this parameter can be unset), and they have to wait at least <code class="docutils literal notranslate"><span class="pre">token_min_interval</span></code> seconds after they used a token before they can use another one (this parameter can be zero). These have no effect in case of infinite tokens.</p>
<p>Having a finite set of both contest- and task-tokens can be very confusing, for the contestants as well as for the contest administrators. Therefore it is common to limit just one type of tokens, setting the other type to be infinite, in order to make the general token availability depend only on the availability of that type (e.g. if you just want to enforce a contest-wide limit on tokens set the contest-token set to be finite and set all task-token sets to be infinite). CWS is aware of this “implementation details” and when one type is infinite it just shows information about the other type, calling it simply “token” (i.e. removing the “contest-” or “task-” prefix).</p>
<p>Note that “token sets” are “intangible”: they’re just a counter shown to the user, computed dynamically every time. Yet, once a token is used, a Token object will be created, stored in the database and associated with the submission it was used on.</p>
<p>Changing token rules during a contest may lead to inconsistencies. Do so at your own risk!</p>
</div>
</div>
<div class="section" id="computation-of-the-score">
<span id="configuringacontest-score"></span><h2>Computation of the score<a class="headerlink" href="#computation-of-the-score" title="Permalink to this headline">¶</a></h2>
<p>The score of a contestant on the contest is always the sum of the score over all tasks. The score on a task depends on the score on each submission via the “score mode” (a setting that can be changed in AdminWebServer for each task).</p>
<div class="section" id="score-modes">
<h3>Score modes<a class="headerlink" href="#score-modes" title="Permalink to this headline">¶</a></h3>
<p>The score mode determines how to compute the score of a contestant in a task from their submissions on that task. There are three score modes, corresponding to the rules of IOI in different years.</p>
<p>“Use best among tokened and last submissions” is the score mode that follows the rules of IOI 2010-2012. It is intended to be used with tasks having some private testcases, and that allow the use of tokens. The score on the task is the best score among “released” submissions. A submission is said to be released if the contestant used a token on it, or if it is the latest one submitted. The idea is that the contestants have to “choose” which submissions they want to use for grading.</p>
<p>“Use best among all submissions” is the score mode that follows the rules of IOI 2013-2016. The score on the task is simply the best score among all submissions.</p>
<p>“Use the sum over each subtask of the best result for that subtask across all submissions” is the score mode that follows the rules of IOI since 2017. It is intended to be used with tasks that have a group score type, like “GroupMin” (note that “group” and “subtask” are synonyms). The score on the task is the sum of the best score for each subtask, over all submissions. The difference with the previous score mode is that here a contestant can achieve the maximum score on the task even when no submission gets the maximum score (for example if each subtask is solved by exactly one submission).</p>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">OutputOnly tasks have a similar behavior to the score mode for IOI 2017-; namely, if a contestant doesn’t submit the output of a testcase, CMS automatically fills in the latest submitted output for that testcase, if present. There is a difference, though: the IOI 2017- score mode would be as if CMS filled the missing output with the one obtaining the highest score, instead of the latest one. Therefore, it might still make sense to use this score mode, even with OutputOnly tasks.</p>
</div>
</div>
<div class="section" id="score-rounding">
<h3>Score rounding<a class="headerlink" href="#score-rounding" title="Permalink to this headline">¶</a></h3>
<p>Based on the ScoreTypes in use and on how they are configured, some submissions may be given a floating-point score. Contest administrators will probably want to show only a small number of these decimal places in the scoreboard. This can be achieved with the <code class="docutils literal notranslate"><span class="pre">score_precision</span></code> fields on the contest and tasks.</p>
<p>The score of a user on a certain task is the maximum among the scores of the “tokened” submissions for that task, and the last one. This score is rounded to a number of decimal places equal to the <code class="docutils literal notranslate"><span class="pre">score_precision</span></code> field of the task. The score of a user on the whole contest is the sum of the <em>rounded</em> scores on each task. This score itself is then rounded to a number of decimal places equal to the <code class="docutils literal notranslate"><span class="pre">score_precision</span></code> field of the contest.</p>
<p>Note that some “internal” scores used by ScoreTypes (for example the subtask score) are not rounded using this procedure. At the moment the subtask scores are always rounded at two decimal places and there’s no way to configure that (note that the score of the submission is the sum of the <em>unrounded</em> scores of the subtasks).</p>
<p>The unrounded score is stored in the database (and it’s rounded only at presentation level) so you can change the <code class="docutils literal notranslate"><span class="pre">score_precision</span></code> at any time without having to rescore any submissions. Yet, you have to make sure that these values are also updated on the RankingWebServers. To do that you can either restart ScoringService or update the data manually (see <a class="reference internal" href="index.html#document-RankingWebServer"><span class="doc">RankingWebServer</span></a> for further information).</p>
</div>
</div>
<div class="section" id="languages">
<h2>Languages<a class="headerlink" href="#languages" title="Permalink to this headline">¶</a></h2>
<div class="section" id="statements">
<h3>Statements<a class="headerlink" href="#statements" title="Permalink to this headline">¶</a></h3>
<p>When there are many statements for a certain task (which are often different translations of the same statement) contest administrators may want to highlight some of them to the users. These may include, for example, the “official” version of the statement (the one that is considered the reference version in case of questions or appeals) or the translations for the languages understood by that particular user. To do that the <code class="docutils literal notranslate"><span class="pre">primary_statements</span></code> field of the tasks and the <code class="docutils literal notranslate"><span class="pre">preferred_languages</span></code> field of the users has to be used.</p>
<p>The <code class="docutils literal notranslate"><span class="pre">primary_statements</span></code> field for the tasks is a list of strings: it specifies the language codes of the statements that will be highlighted to all users. A valid example is <code class="docutils literal notranslate"><span class="pre">en_US,</span> <span class="pre">it</span></code>. The <code class="docutils literal notranslate"><span class="pre">preferred_languages</span></code> field for the users is a list of strings: it specifies the language codes of the statements to highlight. For example <code class="docutils literal notranslate"><span class="pre">de,</span> <span class="pre">de_CH</span></code>.</p>
<p>Note that users will always be able to access all statements, regardless of the ones that are highlighted. Note also that language codes in the form <code class="docutils literal notranslate"><span class="pre">xx</span></code> or <code class="docutils literal notranslate"><span class="pre">xx_YY</span></code> (where <code class="docutils literal notranslate"><span class="pre">xx</span></code> is an <a class="reference external" href="http://www.iso.org/iso/language_codes.htm">ISO 639-1 code</a> and <code class="docutils literal notranslate"><span class="pre">YY</span></code> is an <a class="reference external" href="http://www.iso.org/iso/country_codes.htm">ISO 3166-1 code</a>) will be recognized and presented accordingly. For example <code class="docutils literal notranslate"><span class="pre">en_AU</span></code> will be shown as “English (Australia)”.</p>
</div>
<div class="section" id="interface">
<h3>Interface<a class="headerlink" href="#interface" title="Permalink to this headline">¶</a></h3>
<p>The interface for contestants can be localized (see <a class="reference internal" href="index.html#localization"><span class="std std-ref">Localization</span></a> for how to add new languages), and by default all languages will be available to all contestants. To limit the languages available to the contestants, the field “Allowed localizations” in the contest configuration can be set to the list of allowed language codes. The first of this language codes determines the fallback language in case the preferred language is not available.</p>
</div>
</div>
<div class="section" id="timezone">
<h2>Timezone<a class="headerlink" href="#timezone" title="Permalink to this headline">¶</a></h2>
<p>CMS stores all times as UTC timestamps and converts them to an appropriate timezone when displaying them. This timezone can be specified on a per-user and per-contest basis with the <code class="docutils literal notranslate"><span class="pre">timezone</span></code> field. It needs to contain a string recognized by <a class="reference external" href="http://pytz.sourceforge.net/">pytz</a>, for example <code class="docutils literal notranslate"><span class="pre">Europe/Rome</span></code>.</p>
<p>When CWS needs to show a timestamp to the user it first tries to show it according to the user’s timezone. If the string defining the timezone is unrecognized (for example it is the empty string), CWS will fallback to the contest’s timezone. If it is again unable to interpret that string it will use the local time of the server.</p>
</div>
<div class="section" id="user-login">
<span id="configuringacontest-login"></span><h2>User login<a class="headerlink" href="#user-login" title="Permalink to this headline">¶</a></h2>
<p>Users can log into CWS manually, using their credentials (username and a password), or they can get logged in automatically by CMS based on the IP address their requests are coming from.</p>
<div class="section" id="logging-in-with-ip-based-autologin">
<h3>Logging in with IP-based autologin<a class="headerlink" href="#logging-in-with-ip-based-autologin" title="Permalink to this headline">¶</a></h3>
<p>If the “IP-based autologin” option in the contest configuration is set, CWS tries to find a user that matches the IP address the request is coming from. If it finds exactly one user, the requester is automatically logged in as that user. If zero or more than one user match, CWS does not let the user in (and the incident is logged to allow troubleshooting).</p>
<p>In general, each user can have multiple ranges of IP addresses associated to it. These are defined as a list of subnets in CIDR format (e.g., <cite>192.168.1.0/24</cite>). Only the subnets whose mask is maximal (i.e., <cite>/32</cite> for IPv4 or <cite>/128</cite> for IPv6) are considered for autologin purposes (subnets with non-maximal mask are still useful for IP-based restrictions, see below). The autologin will kick in if <em>any</em> of the subnets matches the IP of the request.</p>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p class="last">If a reverse-proxy (like nginx) is in use then it is necessary to set <code class="docutils literal notranslate"><span class="pre">num_proxies_used</span></code> (in <code class="file docutils literal notranslate"><span class="pre">cms.conf</span></code>) to <code class="docutils literal notranslate"><span class="pre">1</span></code> and configure the proxy in order to properly pass the <code class="docutils literal notranslate"><span class="pre">X-Forwarded-For</span></code>-style headers (see <a class="reference internal" href="index.html#running-cms-recommended-setup"><span class="std std-ref">Recommended setup</span></a>). That configuration option can be set to a higher number if there are more proxies between the origin and the server.</p>
</div>
</div>
<div class="section" id="logging-in-with-credentials">
<h3>Logging in with credentials<a class="headerlink" href="#logging-in-with-credentials" title="Permalink to this headline">¶</a></h3>
<p>If the autologin is not enabled, users can log in with username and password, which have to be specified in the user configuration (in cleartext, for the moment). The password can also be overridden for a specific contest in the participation configuration. These credentials need to be inserted by the admins (i.e. there’s no way to sign up, of log in as a “guest”, etc.).</p>
<p>A successfully logged in user needs to reauthenticate after <code class="docutils literal notranslate"><span class="pre">cookie_duration</span></code> seconds (specified in the <code class="file docutils literal notranslate"><span class="pre">cms.conf</span></code> file) from when they last visited a page.</p>
<p>Even without autologin, it is possible to restrict the IP address or subnet that the user is using for accessing CWS, using the “IP-based login restriction” option in the contest configuration (in which case, admins need to set <code class="docutils literal notranslate"><span class="pre">num_proxies_used</span></code> as before). If this is set, then the login will fail if the IP address that attempted it does not match at least one of the addresses or subnets specified in the participation settings. If the participation IP address is not set, then no restriction applies.</p>
</div>
<div class="section" id="failure-to-login">
<h3>Failure to login<a class="headerlink" href="#failure-to-login" title="Permalink to this headline">¶</a></h3>
<p>The following are some common reasons for login failures, all of them coming with some useful log message from CWS.</p>
<ul class="simple">
<li>IP address mismatch (with IP-based autologin): if the IP address doesn’t match any subnet of any participation or if it matches some subnets of more than one participation, then the login fails. Note that if the user is using the IP address of a different user, CWS will happily log them in without noticing anything.</li>
<li>IP address mismatch (using IP-based login restrictions): the login fails if the request comes from an IP address that doesn’t match any of the participation’s IP subnets (non-maximal masks are taken into consideration here).</li>
<li>Blocked hidden participations: users whose participation is hidden cannot log in if “Block hidden participations” is set in the contest configuration.</li>
</ul>
</div>
</div>
<div class="section" id="usaco-like-contests">
<span id="configuringacontest-usaco-like-contests"></span><h2>USACO-like contests<a class="headerlink" href="#usaco-like-contests" title="Permalink to this headline">¶</a></h2>
<p>One trait of the <a class="reference external" href="http://usaco.org/">USACO</a> contests is that the contests themselves are many days long but each user is only able to compete for a few hours after their first login (after that they are not able to send any more submissions). This can be done in CMS too, using the <code class="docutils literal notranslate"><span class="pre">per_user_time</span></code> field of contests. If it is unset the contest will behave “normally”, that is all users will be able to submit solutions from the contest’s beginning until the contest’s end. If, instead, <code class="docutils literal notranslate"><span class="pre">per_user_time</span></code> is set to a positive integer value, then a user will only have a limited amount of time. In particular, after they log in, they will be presented with an interface similar to the pre-contest one, with one additional “start” button. Clicking on this button starts the time frame in which the user can compete (i.e. read statements, download attachments, submit solutions, use tokens, send user tests, etc.). This time frame ends after <code class="docutils literal notranslate"><span class="pre">per_user_time</span></code> seconds or when the contest <code class="docutils literal notranslate"><span class="pre">stop</span></code> time is reached, whichever comes first. After that the interface will be identical to the post-contest one: the user won’t be able to do anything. See <a class="reference external" href="https://github.com/cms-dev/cms/issues/61">issue #61</a>.</p>
<p>The time at which the user clicks the “start” button is recorded in the <code class="docutils literal notranslate"><span class="pre">starting_time</span></code> field of the user. You can change that to shift the user’s time frame (but we suggest to use <code class="docutils literal notranslate"><span class="pre">extra_time</span></code> for that, explained in <a class="reference internal" href="#configuringacontest-extra-time"><span class="std std-ref">Extra time and delay time</span></a>) or unset it to make the user able to start its time frame again. Do so at your own risk!</p>
</div>
<div class="section" id="extra-time-and-delay-time">
<span id="configuringacontest-extra-time"></span><h2>Extra time and delay time<a class="headerlink" href="#extra-time-and-delay-time" title="Permalink to this headline">¶</a></h2>
<p>Contest administrators may want to give some users a short additional amount of time in which they can compete to compensate for an incident (e.g. a hardware failure) that made them unable to compete for a while during the “intended” time frame. That’s what the <code class="docutils literal notranslate"><span class="pre">extra_time</span></code> field of the users is for. The time frame in which the user is allowed to compete is expanded by its <code class="docutils literal notranslate"><span class="pre">extra_time</span></code>, even if this would lead the user to be able to submit after the end of the contest.</p>
<p>During extra time the user will continue to receive newly generated tokens. If you don’t want them to have more tokens that other contestants, set the <code class="docutils literal notranslate"><span class="pre">token_max_number</span></code> parameter described above to the number of tokens you expect a user to have at their disposal during the whole contest (if it doesn’t already have a value less than or equal to this).</p>
<p>Contest administrators can also alter the competition time of a contestant setting <code class="docutils literal notranslate"><span class="pre">delay_time</span></code>, which has the effect of translating the competition time window for that contestant of the specified numer of seconds in the future. Thus, while setting <code class="docutils literal notranslate"><span class="pre">extra_time</span></code> <em>adds</em> some times at the end of the contest, setting <code class="docutils literal notranslate"><span class="pre">delay_time</span></code> <em>moves</em> the whole time window. As for <code class="docutils literal notranslate"><span class="pre">extra_time</span></code>, setting <code class="docutils literal notranslate"><span class="pre">delay_time</span></code> may extend the contestant time window beyond the end of the contest itself.</p>
<p>Both options have to be set to a non negative number. They can be used together, producing both their effects. Please read <a class="reference internal" href="index.html#document-Detailed timing configuration"><span class="doc">Detailed timing configuration</span></a> for a more in-depth discussion of their exact effect.</p>
<p>Note also that submissions sent during the extra time will continue to be considered when computing the score, even if the <code class="docutils literal notranslate"><span class="pre">extra_time</span></code> field of the user is later reset to zero (for example in case the user loses the appeal): you need to completely delete them from the database or make them unofficial, and make sure the score in all rankings reflects the new state.</p>
</div>
<div class="section" id="analysis-mode">
<h2>Analysis mode<a class="headerlink" href="#analysis-mode" title="Permalink to this headline">¶</a></h2>
<p>After the contest it is often customary to allow contestants to see the results of all their submissions and use the grading system to try different solutions. CMS offers an analysis mode to do this. Solutions submitted during the analysis are evaluated as usual, but are marked as not official, and thus do not contribute to the rankings. Users will also be prevented from using tokens.</p>
<p>The admins can enable the analysis mode in the contest configuration page in AWS; they also must set start end stop time (which must be after the contest end).</p>
<p>By awarding extra time or adding delay to a contestant, it is possible to extend the contest time for a user over the start of the analysis. In this case, the start of the analysis will be postponed for this user. If the contest rules contemplate extra time or delay, we suggest to avoid starting the analysis right after the end of the contest.</p>
</div>
<div class="section" id="programming-languages">
<span id="configuringacontest-programming-languages"></span><h2>Programming languages<a class="headerlink" href="#programming-languages" title="Permalink to this headline">¶</a></h2>
<p>CMS allows to restrict the set of programming languages available to contestants in a certain contest; the configuration is in the contest page in AWS.</p>
<p>CMS offers out of the box the following combination of languages: C, C++, Pascal, Java (using a JDK), Python 2 and 3, PHP, Haskell, Rust, C#.</p>
<p>C, C++ and Pascal are the default languages, and have been tested thoroughly in many contests.</p>
<p>PHP and Python have only been tested with Batch task types, and have not thoroughly analyzed for potential security and usability issues. Being run under the sandbox, they should be reasonably safe, but, for example, the libraries available to contestants might be hard to control.</p>
<p>Java works with Batch and Communication task types. Under usual conditions (default submission format) contestants must name their class as the short name of the task.</p>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p>Java uses multithreading even for simple programs. Therefore, if this language is allowed in the contest, multithreading and multiprocessing will be allowed in the sandbox for <em>all</em> evaluations (even with other languages).</p>
<p class="last">If a solution uses multithreading or multiprocessing, the time limit is checked against the sum of the user times of all threads and processes.</p>
</div>
<div class="section" id="language-details">
<h3>Language details<a class="headerlink" href="#language-details" title="Permalink to this headline">¶</a></h3>
<ul class="simple">
<li>C and C++ are supported through the GNU Compiler Collection. Submissions are optimized with <code class="docutils literal notranslate"><span class="pre">-O2</span></code>. Multiple C and C++ language revisions are supported.</li>
<li>Java uses the system version of the Java compiler and JVM.</li>
<li>Pascal support is provided by <code class="docutils literal notranslate"><span class="pre">fpc</span></code>, and submissions are optimized with <code class="docutils literal notranslate"><span class="pre">-O2</span></code>.</li>
<li>Python submissions are executed using the system Python interpreter (you need to have <code class="docutils literal notranslate"><span class="pre">/usr/bin/python2</span></code> or <code class="docutils literal notranslate"><span class="pre">/usr/bin/python3</span></code>, respectively).</li>
<li>PHP submissions are interpreted by <code class="docutils literal notranslate"><span class="pre">/usr/bin/php</span></code>.</li>
<li>Haskell support is provided by <code class="docutils literal notranslate"><span class="pre">ghc</span></code>, and submissions are optimized with <code class="docutils literal notranslate"><span class="pre">-O2</span></code>.</li>
<li>Rust support is provided by <code class="docutils literal notranslate"><span class="pre">rustc</span></code>, and submissions are optimized with <code class="docutils literal notranslate"><span class="pre">-O</span></code>.</li>
<li>C# uses the system version of the Mono compiler <code class="docutils literal notranslate"><span class="pre">mcs</span></code> and the runtime <code class="docutils literal notranslate"><span class="pre">mono</span></code>. Submissions are optimized with <code class="docutils literal notranslate"><span class="pre">-optimize+</span></code>.</li>
</ul>
</div>
<div class="section" id="custom-languages">
<h3>Custom languages<a class="headerlink" href="#custom-languages" title="Permalink to this headline">¶</a></h3>
<p>Additional languages can be defined if necessary. This works in the same way <a class="reference internal" href="index.html#tasktypes-custom"><span class="std std-ref">as with task types</span></a>: the classes need to extend <code class="xref py py-class docutils literal notranslate"><span class="pre">cms.grading.language.Language</span></code> and the entry point group is called <cite>cms.grading.languages</cite>.</p>
</div>
</div>
</div>
<span id="document-Detailed timing configuration"></span><div class="section" id="detailed-timing-configuration">
<h1>Detailed timing configuration<a class="headerlink" href="#detailed-timing-configuration" title="Permalink to this headline">¶</a></h1>
<p>This section describes the exact meaning of CMS parameters for
controlling the time window allocated to each contestant. Please see
<a class="reference internal" href="index.html#document-Configuring a contest"><span class="doc">Configuring a contest</span></a> for a more gentle introduction and the
intended usage of the various parameters.</p>
<p>When setting up a contest, you will need to decide the time window in
which contestants will be able to interact with the contest (by
reading statements, submit solutions, …). In CMS there are several
parameters that allow to control this time window, and it is also
possible to personalize it for each single user in case it is needed.</p>
<p>The first decision to chose among these two possibilities:</p>
<ol class="arabic simple">
<li>all contestants will start and end the contest at the same time
(unless otherwise decided by the admins during the contest for
fairness reasons);</li>
<li>each contestant will start the contest at the time they decide.</li>
</ol>
<p>The first situation is that we will refer to as a fixed-window
contest, whereas we will refer to the second situation as
customized-window contest.</p>
<div class="section" id="fixed-window-contests">
<h2>Fixed-window contests<a class="headerlink" href="#fixed-window-contests" title="Permalink to this headline">¶</a></h2>
<p>These are quite simple to configure: you just need to set
<code class="docutils literal notranslate"><span class="pre">start_time</span></code> and <code class="docutils literal notranslate"><span class="pre">end_time</span></code>, and by default all users will be able
to interact with the contest between these two instants.</p>
<p>For fairness reasons, during the contest you may want to extend the
time window for all or for particular users. In the first case, you
just need to change the end_time parameter. In the latter case, you
can use one of two slightly different per-contestant parameters:
<code class="docutils literal notranslate"><span class="pre">extra_time</span></code> and <code class="docutils literal notranslate"><span class="pre">delay_time</span></code>.</p>
<p>You can use <code class="docutils literal notranslate"><span class="pre">extra_time</span></code> to award more time at the end of the
contest for a specific contestant, whereas you can use <code class="docutils literal notranslate"><span class="pre">delay_time</span></code>
to shift in the future the time window of the contest just for that
user. There are two main practical differences between these two
options.</p>
<ol class="arabic simple">
<li>If you set <code class="docutils literal notranslate"><span class="pre">extra_time</span></code> to S seconds, the contestant will be able
to interact with the contest in the first S seconds of it, whereas
if you use <code class="docutils literal notranslate"><span class="pre">delay_time</span></code>, they will not, as in the first case the
time window is extended, in the second is shifted (if S seconds
have already passed from the start of the contest, then there is no
difference).</li>
<li>If tokens are generated every M minutes, and you set <code class="docutils literal notranslate"><span class="pre">extra_time</span></code>
to S seconds, then tokens for that contestants are generated at
<code class="docutils literal notranslate"><span class="pre">start_time</span></code> + k*M (in particular, it might be possible that more
tokens are generated for contestants with <code class="docutils literal notranslate"><span class="pre">extra_time</span></code>); if
instead you set <code class="docutils literal notranslate"><span class="pre">delay_time</span></code> to S seconds, tokens for that
contestants are generated at start_time + S + k*M (i.e., they are
shifted from the original, and the same amount of tokens as other
contestants will be generated).</li>
</ol>
<p>If needed, it is possible to use both at the same time.</p>
</div>
<div class="section" id="customized-window-contests">
<h2>Customized-window contests<a class="headerlink" href="#customized-window-contests" title="Permalink to this headline">¶</a></h2>
<p>In these contests, contestants can use a time window of fixed length
(<code class="docutils literal notranslate"><span class="pre">per_user_time</span></code>), starting from the first time they log in between
<code class="docutils literal notranslate"><span class="pre">start_time</span></code> and <code class="docutils literal notranslate"><span class="pre">end_time</span></code>. Moreover, the time window is capped at
<code class="docutils literal notranslate"><span class="pre">end_time</span></code> (so if <code class="docutils literal notranslate"><span class="pre">per_user_time</span></code> is 5 hours and a contestant logs
in for the first time one minute before <code class="docutils literal notranslate"><span class="pre">end_time</span></code>, they will have
just one minute).</p>
<p>Again, admins can change the time windows of specific contestants for
fairness reasons. In addition to <code class="docutils literal notranslate"><span class="pre">extra_time</span></code> and <code class="docutils literal notranslate"><span class="pre">delay_time</span></code>,
they can also use <code class="docutils literal notranslate"><span class="pre">starting_time</span></code>, which is automatically set by CMS
when the contestant logs in for the first time.</p>
<p>The meaning of <code class="docutils literal notranslate"><span class="pre">extra_time</span></code> is to extend both the contestant
time window (as defined by <code class="docutils literal notranslate"><span class="pre">starting_time</span></code> + <code class="docutils literal notranslate"><span class="pre">per_user_time</span></code>) and
the contest time window (as defined by <code class="docutils literal notranslate"><span class="pre">end_time</span></code>) by the value of
<code class="docutils literal notranslate"><span class="pre">extra_time</span></code>, but only for that contestant. Therefore, setting
<code class="docutils literal notranslate"><span class="pre">extra_time</span></code> to S seconds effectively allows a contestant to use S
seconds more than before (regardless of the time they started the
contest).</p>
<p>Again, delay time is similar, but it shifts both contestant and
contest time window by that value. The effect on available time
similar to that achieved by setting <code class="docutils literal notranslate"><span class="pre">extra_time</span></code>, with the
difference explained before in point 1. Also, there is a difference in
token generation as explained in point 2 above.</p>
<p>Finally, changing <code class="docutils literal notranslate"><span class="pre">starting_time</span></code> is very similar to changing
<code class="docutils literal notranslate"><span class="pre">delay_time</span></code>, but it shifts just the contestant time window, hence
if that window was already going over <code class="docutils literal notranslate"><span class="pre">end_time</span></code>, at all effects
advancing <code class="docutils literal notranslate"><span class="pre">starting_time</span></code> would not award more time to the
contestant, because the end would still be capped at <code class="docutils literal notranslate"><span class="pre">end_time</span></code>. The
effect on token generation is the same.</p>
<p>There is probably no need to fiddle with more than one of these
three parameters, and our suggestion is to just use <code class="docutils literal notranslate"><span class="pre">extra_time</span></code> or
<code class="docutils literal notranslate"><span class="pre">delay_time</span></code> to award more time to a contestant.</p>
</div>
</div>
<span id="document-Task types"></span><div class="section" id="task-types">
<h1>Task types<a class="headerlink" href="#task-types" title="Permalink to this headline">¶</a></h1>
<div class="section" id="introduction">
<h2>Introduction<a class="headerlink" href="#introduction" title="Permalink to this headline">¶</a></h2>
<p>In the CMS terminology, the task type of a task describes how to compile and evaluate the submissions for that task. In particular, they may require additional files called managers, provided by the admins.</p>
<p>A submission goes through two steps involving the task type: the compilation, that usually creates an executable from the submitted files, and the evaluation, that runs this executable against the set of testcases and produces an outcome for each of them.</p>
<p>Note that the outcome doesn’t need to be obviously tied to the score for the submission: typically, the outcome is computed by a grader (which is an executable or a program stub passed to CMS) or a comparator (a program that decides if the output of the contestant’s program is correct) and not directly by the task type. Hence, the task type doesn’t need to know the meaning of the outcome, which is instead known by the grader and by the <a class="reference internal" href="index.html#document-Score types"><span class="doc">score type</span></a>.</p>
<p>An exception to this is when the contestant’s source fails (for example, exceeding the time limit); in this case the task type will assign directly an outcome, usually 0.0; admins must consider that when planning the outcomes for a task.</p>
</div>
<div class="section" id="standard-task-types">
<h2>Standard task types<a class="headerlink" href="#standard-task-types" title="Permalink to this headline">¶</a></h2>
<p>CMS ships with four task types: Batch, OutputOnly, Communication, TwoSteps. The first three are well tested and reasonably strong against cheating attempts and stable with respect to the evaluation times. TwoSteps is a somewhat simpler way to implement a special case of a Communication task, but it is substantially less secure with respect to cheating. We suggest avoiding TwoSteps for new tasks, and migrating old tasks to Communication.</p>
<p>OutputOnly does not involve programming languages. Batch is tested with all languages CMS supports out of the box, (C, C++, Pascal, Java, C#, Python, PHP, Haskell, Rust), but only with the first five when using a grader. Communication is tested with C, C++, Pascal and Java. TwoSteps only with C. Regardless, with some work all task types should work with all languages.</p>
<p>Task types may have parameters to configure their behaviour (for example, a parameter for Batch defines whether to use a simple diff or a checker to evaluate the output). You can set these parameters, for each task, on the task’s page in AdminWebServer.</p>
<div class="section" id="batch">
<span id="tasktypes-batch"></span><h3>Batch<a class="headerlink" href="#batch" title="Permalink to this headline">¶</a></h3>
<p>In a Batch task, each testcase has an input (usually kept secret from the contestants), and the contestant’s solution must produce a correct output for that input.</p>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p class="last">If the input, or part thereof, is supposed to be a secret from the contestant’s code at least for part of the evaluation, then Batch is insecure and Communication should be used.</p>
</div>
<p>The contestant must submit a single source file; thus the submission format should contain one element with a <code class="docutils literal notranslate"><span class="pre">.%l</span></code> placeholder for the language extension.</p>
<p>Batch has three parameters:</p>
<ul class="simple">
<li>the first specifies whether the source submitted by the contestant is compiled on its own, or together with a grader provided by the admins;</li>
<li>the second specifies the filenames of input and output (for reading and writing by the contestant source or by the grader), or whether to redirect them to standard input and standard output (if left blank);</li>
<li>the third whether to compare correct output and contestant-produced output with a simple diff, or with an admin-provided comparator.</li>
</ul>
<p>A grader is a source file that is compiled with the contestant’s source, and usually performs I/O for the contestants, so that they only have to implement one or more functions. If the task uses a grader, the admins must provide a manager called <code class="file docutils literal notranslate"><span class="pre">grader.</span><em><span class="pre">ext</span></em></code> for each allowed language, where <code class="file docutils literal notranslate"><em><span class="pre">ext</span></em></code> is the standard extension of a source file in that language. If header files are needed, they can be provided as additional managers with an appropriate extension (for example, <code class="docutils literal notranslate"><span class="pre">.h</span></code> for C/C++ and <code class="docutils literal notranslate"><span class="pre">lib.pas</span></code> for Pascal).</p>
<p>The output produced by the contestant (possibly through the grader) is then evaluated against the correct output. This can be done with <a class="reference internal" href="#tasktypes-white-diff"><span class="std std-ref">white-diff</span></a>, or using a <a class="reference internal" href="#tasktypes-checker"><span class="std std-ref">comparator</span></a>. In the latter case, the admins must provide an executable manager called <code class="file docutils literal notranslate"><span class="pre">checker</span></code>. If the contestant’s code fails, this step is omitted, and the outcome will be 0.0 and the message will explain the reason.</p>
<p>Batch supports user tests; if a grader is used, the contestants must provide their own grader (a common practice is to provide a simple grader to contestants, that can be used for local testing and for server-side user tests). The output produced by the contestant’s solution, possibly through the grader, is sent back to the contestant; it is not evaluated against a correct output.</p>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">Batch tasks are supported also for Java, with some requirements. The top-level class in the contestant’s source must be named like the short name of the task. The one in the grader (containing the main method) must be  named <code class="docutils literal notranslate"><span class="pre">grader</span></code>.</p>
</div>
</div>
<div class="section" id="outputonly">
<span id="tasktypes-outputonly"></span><h3>OutputOnly<a class="headerlink" href="#outputonly" title="Permalink to this headline">¶</a></h3>
<p>In an OutputOnly task, contestants can see the input of each testcase, and have to compute offline a correct output.</p>
<p>In any submission, contestants may submit outputs for any subset of testcases. The submission format therefore must contain one element for each testcase, and the elements must be of the form <code class="file docutils literal notranslate"><span class="pre">output_</span><em><span class="pre">codename</span></em><span class="pre">.txt</span></code> where <code class="samp docutils literal notranslate"><em><span class="pre">codename</span></em></code> is the codename for the testcase.</p>
<p>Moreover, CMS will automatically fill the missing files in the current submission with those in the previous one, as if the contestant had submitted them. For example, if there were 4 testcases, and the following submissions:</p>
<ul class="simple">
<li>submission s1 with files f1 and f2,</li>
<li>submission s2 with files f2’ and f3,</li>
<li>submission s3 with file f4,</li>
</ul>
<p>then s1 will be judged using f1 and f2; s2 will be judged using f1, f2’ and f3; and finally s3 will be judged using f1, f2’, f3 and f4.</p>
<p>OutputOnly has one parameter, that specifies whether to compare correct output and contestant-produced output with <a class="reference internal" href="#tasktypes-white-diff"><span class="std std-ref">white-diff</span></a>, or using a <a class="reference internal" href="#tasktypes-checker"><span class="std std-ref">comparator</span></a> (exactly the same as the third parameter for Batch). In the latter case, the admins must provide an executable manager called <code class="file docutils literal notranslate"><span class="pre">checker</span></code>.</p>
</div>
<div class="section" id="communication">
<span id="tasktypes-communication"></span><h3>Communication<a class="headerlink" href="#communication" title="Permalink to this headline">¶</a></h3>
<p>Communication tasks are similar to Batch tasks, but should be used when the input, or part of it, must remain secret, at least for some time, to the contestant’s code. This is the case, for example, in tasks where the contestant’s code must ask questions about the input; or when it must compute the solution incrementally after seeing partial views of the input.</p>
<p>In practice, Communication tasks have two processes, running in two different sandboxes:</p>
<ul class="simple">
<li>the first (called manager) is entirely controlled by the admins; it reads the input, communicates with the other one, and writes a <a class="reference internal" href="#tasktypes-standard-manager-output"><span class="std std-ref">standard manager output</span></a>;</li>
<li>the second is where the contestant’s code runs, optionally after being compiled together with an admin-provided stub that helps with the communication with the first process; it doesn’t have access to the input, just to what the manager communicates.</li>
</ul>
<p>This setup ensures that the contestant’s code cannot access forbidden data, even in the case they have full knowledge of the admin code.</p>
<p>The admins must provide an executable manager called <code class="docutils literal notranslate"><span class="pre">manager</span></code>. It can read the testcase input from stdin, and will also receive as argument the filenames of two FIFOs, from and to the contestant process (in this order). It must write to stdout the outcome and to stderr the message for the contestant (see <a class="reference internal" href="#tasktypes-standard-manager-output"><span class="std std-ref">details about the format`</span></a>). If the contestant’s process fails, the output of the manager is ignored, and the outcome will be 0.0 and the message will explain the reason.</p>
<p>Admins can also provide a manager called <code class="file docutils literal notranslate"><span class="pre">stub.</span><em><span class="pre">ext</span></em></code> for each allowed language, where <code class="file docutils literal notranslate"><em><span class="pre">ext</span></em></code> is the standard extension of a source file in that language. The task type can be set up to compile the stub with the contestant’s source. Usually, a stub takes care of the communication with the manager, so that the contestants have to implement only a function. As for Batch, admins can also add header file that will be used when compiling the stub and the contestant’s source.</p>
<p>The contestant’s program, regardless of whether it’s compiled with or without a stub, can be set up to communicate with the manager in two ways: through the standard input and output, or through FIFOs (in which case the FIFOs’ filenames will be given as arguments, first the one from the manager and then the one to it).</p>
<p>The first parameter of the task type controls the number of user processes. If it is equal to 1, the behavior will be as explained above. If it is an integer N greater than 1, there are a few differences:</p>
<ul class="simple">
<li>there will be N processes with the contestant’s code and the stub (if present) running;</li>
<li>there will be N pairs of FIFOs, one for each process running the contestant’s program; the manager will receive as argument all pairs in order, and each contestant program will receive its own (as arguments or redirected through stdin/stdout);</li>
<li>each copy of the contestant’s program will receive as an additional argument its 0-based index within the running programs;</li>
<li>the time limit is checked against the total user time of all the contestant’s processes.</li>
</ul>
<p>The submission format must contain one or more filenames ending with <code class="docutils literal notranslate"><span class="pre">.%l</span></code>. Multiple source files are simply linked together. Usually the number of files to submit is equal to the number of processes.</p>
<p>Communication supports user tests. In addition to the input file, contestant must provide the stub and their source file. The admin-provided manager will be used; the output returned to the contestant will be what the manager writes to the file <code class="file docutils literal notranslate"><span class="pre">output.txt</span></code>.</p>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">Particular care must be taken for tasks where the communication through the FIFOs is particularly large or frequent. In these cases, the time to send the data may dominate the actual algorithm runtime, thus making it hard to distinguish between different complexities.</p>
</div>
</div>
<div class="section" id="twosteps">
<h3>TwoSteps<a class="headerlink" href="#twosteps" title="Permalink to this headline">¶</a></h3>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p class="last">This task type is not secure; the user source could intercept the main function and take control of input reading and communication between the processes, which is not monitored. Admins should use Communication instead.</p>
</div>
<p>In a TwoSteps task, contestants submit two source files implementing a function each (the idea is that the first function gets the input and compute some data from it with some restriction, and the second tries to retrieve the original data).</p>
<p>The admins must provide a manager, which is compiled together with both of the contestant-submitted files. The manager needs to be named <code class="file docutils literal notranslate"><span class="pre">manager.</span><em><span class="pre">ext</span></em></code>, where <code class="docutils literal notranslate"><span class="pre">{ext}</span></code> is the standard extension of a source file in that language. Furthermore, the admins must provide appropriate header files for the two source files and for the manager, even if they are empty.</p>
<p>The resulting executable is run twice (one acting as the computer, one acting as the retriever). The manager in the computer executable must take care of reading the input from standard input; the one in the retriever executable of writing the retrieved data to standard output. Both must take responsibility of the communication between them through a pipe.</p>
<p>More precisely, the executable is called with two arguments: the first is an integer which is 0 if the executable is the computer, and 1 if it is the retriever; the second is the name of the pipe to be used for communication between the processes.</p>
<p>TwoSteps has one parameter, similar to Batch’s third, that specifies whether to compare the second process output with the correct output using white-diff or a checker. In the latter case, an executable manager named <code class="file docutils literal notranslate"><span class="pre">checker</span></code> must be provided.</p>
<p>TwoSteps supports user tests; contestants must provide the manager in addition to the input and their sources.</p>
<p><strong>How to migrate from TwoSteps to Communication.</strong> Any TwoSteps task can be implemented as a Communication task with two processes. The functionalities in the stub should be migrated to Communication’s manager, which also must enforce any restriction in the computed data.</p>
</div>
</div>
<div class="section" id="white-diff-comparator">
<span id="tasktypes-white-diff"></span><h2>White-diff comparator<a class="headerlink" href="#white-diff-comparator" title="Permalink to this headline">¶</a></h2>
<p>White-diff is the only built-in comparator. It can be used when each testcase has a unique correct output file, up to whitespaces. White-diff will report an outcome of 1.0 if the correct output and the contestant’s output match up to whitespaces, or 0.0 if they don’t.</p>
<p>More precisely, white-diff will return that a pair of files match if all of these conditions are satisfied:</p>
<ul class="simple">
<li>they have the same number of lines (apart from trailing lines composed only of whitespaces, which are ignored);</li>
<li>for each corresponding line in the two files, the list of non-empty, whitespace-separated tokens is the same (in particular, tokens appear in the same order).</li>
</ul>
<p>It treats as whitespace any repetition of these characters: space, newline, carriage return, tab, vertical tab, form feed.</p>
<p>Note that spurious empty lines in the middle of an output will make white-diff report a no-match, even if all tokens are correct.</p>
</div>
<div class="section" id="checker">
<span id="tasktypes-checker"></span><h2>Checker<a class="headerlink" href="#checker" title="Permalink to this headline">¶</a></h2>
<p>When there are multiple correct outputs, or when there is partial scoring, white-diff is not powerful enough. In this cases, a checker can be used to perform a complex validation. It is an executable manager, usually named <code class="file docutils literal notranslate"><span class="pre">checker</span></code>.</p>
<p>It will receive as argument three filenames, in order: input, correct output, and contestant’s output. It will then write a <a class="reference internal" href="#tasktypes-standard-manager-output"><span class="std std-ref">standard manager output</span></a> to stdout and stderr.</p>
<p>It is preferred to compile the checker statically (e.g., with <code class="docutils literal notranslate"><span class="pre">-static</span></code> using <code class="docutils literal notranslate"><span class="pre">gcc</span></code> or <code class="docutils literal notranslate"><span class="pre">g++</span></code>) to avoid potential problems with the sandbox.</p>
</div>
<div class="section" id="standard-manager-output">
<span id="tasktypes-standard-manager-output"></span><h2>Standard manager output<a class="headerlink" href="#standard-manager-output" title="Permalink to this headline">¶</a></h2>
<p>A standard manager output is a format that managers can follow to write an outcome and a message for the contestant.</p>
<p>To follow the standard manager output, a manager must write on stdout a single line, containing a floating point number, the outcome; it must write to stderr a single line containing the message for the contestant. Following lines to stdout or stderr will be ignored.</p>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">If the manager writes to standard error the special strings “translate:success”, “translate:wrong” or “translate:partial”, these will be respectively shown to the contestants as the localized messages for “Output is correct”, “Output isn’t correct”, and “Output is partially correct”.</p>
</div>
</div>
<div class="section" id="custom-task-types">
<span id="tasktypes-custom"></span><h2>Custom task types<a class="headerlink" href="#custom-task-types" title="Permalink to this headline">¶</a></h2>
<p>If the set of default task types doesn’t suit a particular need, a custom task type can be provided. For that, in a separate “workspace” (i.e., a directory disjoint from CMS’s tree), write a new Python class that extends <code class="xref py py-class docutils literal notranslate"><span class="pre">cms.grading.tasktypes.TaskType</span></code> and implements its abstract methods. The docstrings of those methods explain what they need to do, and the default task types can provide examples.</p>
<p>An accompanying <code class="file docutils literal notranslate"><span class="pre">setup.py</span></code> file must also be prepared, which must reference the task type’s class as an “entry point”: the <code class="docutils literal notranslate"><span class="pre">entry_points</span></code> keyword argument of the <code class="docutils literal notranslate"><span class="pre">setup</span></code> function, which is a dictionary, needs to contain a key named <code class="docutils literal notranslate"><span class="pre">cms.grading.tasktypes</span></code> whose value is a list of strings; each string represents an entry point in the format <code class="docutils literal notranslate"><span class="pre">{name}={package.module}:{Class}</span></code>, where <code class="docutils literal notranslate"><span class="pre">{name}</span></code> is the name of the entry point (at the moment it plays no role for CMS, but please name it in the same way as the class) and <code class="docutils literal notranslate"><span class="pre">{package.module}</span></code> and <code class="docutils literal notranslate"><span class="pre">{Class}</span></code> are the full module name and the name of the class for the task type.</p>
<p>A full example of <code class="file docutils literal notranslate"><span class="pre">setup.py</span></code> is as follows:</p>
<div class="highlight-python notranslate"><div class="highlight"><pre><span></span><span class="kn">from</span> <span class="nn">setuptools</span> <span class="kn">import</span> <span class="n">setup</span><span class="p">,</span> <span class="n">find_packages</span>

<span class="n">setup</span><span class="p">(</span>
    <span class="n">name</span><span class="o">=</span><span class="s2">&quot;my_task_type&quot;</span><span class="p">,</span>
    <span class="n">version</span><span class="o">=</span><span class="s2">&quot;1.0&quot;</span><span class="p">,</span>
    <span class="n">packages</span><span class="o">=</span><span class="n">find_packages</span><span class="p">(),</span>
    <span class="n">entry_points</span><span class="o">=</span><span class="p">{</span>
        <span class="s2">&quot;cms.grading.tasktypes&quot;</span><span class="p">:</span> <span class="p">[</span>
            <span class="s2">&quot;MyTaskType=my_package.my_module:MyTaskType&quot;</span>
        <span class="p">]</span>
    <span class="p">}</span>
<span class="p">)</span>
</pre></div>
</div>
<p>Once that is done, install the distribution by executing</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>python3 setup.py install
</pre></div>
</div>
<p>CMS needs to be restarted for it to pick up the new task type.</p>
<p>For additional information see the <a class="reference external" href="https://docs.python.org/3/distutils/setupscript.html">general distutils documentation</a> and the <a class="reference external" href="https://setuptools.readthedocs.io/en/latest/setuptools.html#dynamic-discovery-of-services-and-plugins">section of the setuptools documentation about entry points</a>.</p>
</div>
</div>
<span id="document-Score types"></span><div class="section" id="score-types">
<h1>Score types<a class="headerlink" href="#score-types" title="Permalink to this headline">¶</a></h1>
<div class="section" id="introduction">
<h2>Introduction<a class="headerlink" href="#introduction" title="Permalink to this headline">¶</a></h2>
<p>For every submission, the score type of a task comes into play after the <a class="reference internal" href="index.html#document-Task types"><span class="doc">task type</span></a> produced an outcome for each testcase. Indeed, the most important duty of the score type is to describe how to translate the list of outcomes into a single number: the score of the submission. The score type also produces a more informative output for the contestants, and the same information (score and detail) for contestants that did not use a token on the submission. In CMS, these latter set of information is called public, since the contestant can see them without using any tokens.</p>
</div>
<div class="section" id="standard-score-types">
<h2>Standard score types<a class="headerlink" href="#standard-score-types" title="Permalink to this headline">¶</a></h2>
<p>CMS ships with the following score types: Sum, GroupMin, GroupMul, GroupThreshold.</p>
<p>The first of the four well-tested score types, Sum, is the simplest you can imagine, just assigning a fixed amount of points for each correct testcase. The other three are useful for grouping together testcases and assigning points for that group only if some conditions held. Groups are also known as subtasks in some contests. The group score types also allow test cases to be weighted, even for groups of size 1.</p>
<p>Also like task types, the behavior of score types is configurable from the task’s page in AdminWebServer.</p>
<div class="section" id="sum">
<span id="scoretypes-sum"></span><h3>Sum<a class="headerlink" href="#sum" title="Permalink to this headline">¶</a></h3>
<p>This score type interprets the outcome for each testcase as a floating-point number measuring how good the submission was in solving that testcase, where 0.0 means that the submission failed, and 1.0 that it solved the testcase correctly. The score of that submission will be the sum of all the outcomes for each testcase, multiplied by an integer parameter given in the Score type parameter field in AdminWebServer. The parameter field must contain only this integer. The public score is given by the same computation over the public testcases instead of over all testcases.</p>
<p>For example, if there are 20 testcases, 2 of which are public, and the parameter string is <code class="docutils literal notranslate"><span class="pre">5</span></code>, a correct solution will score 100 points (20 times 5) out of 100, and its public score will be 10 points (2 times 5) out of 10.</p>
</div>
<div class="section" id="groupmin">
<span id="scoretypes-groupmin"></span><h3>GroupMin<a class="headerlink" href="#groupmin" title="Permalink to this headline">¶</a></h3>
<p>With the GroupMin score type, outcomes are again treated as a measure of correctness, from 0.0 (incorrect) to 1.0 (correct); testcases are split into groups, and each group has an integral multiplier. The score is the sum of the score of each group, which in turn is the minimum outcome within the group times the multiplier. The public score is computed over all groups whose testcases are all public.</p>
<p>The parameters string for GroupMin is of the form <code class="samp docutils literal notranslate"><span class="pre">[[</span><em><span class="pre">m1</span></em><span class="pre">,</span> <em><span class="pre">t1</span></em><span class="pre">],</span> <span class="pre">[</span><em><span class="pre">m2</span></em><span class="pre">,</span> <em><span class="pre">t2</span></em><span class="pre">],</span> <span class="pre">...]</span></code>, where for each pair the first element is the multiplier, and the second is either always an integer, or always a string.</p>
<p>In the first case (second element is always an integer), the integer is interpreted as the number of testcases that the group consumes (ordered by codename). That is, the first group comprises the first <code class="samp docutils literal notranslate"><em><span class="pre">t1</span></em></code> testcases and has multiplier <code class="samp docutils literal notranslate"><em><span class="pre">m1</span></em></code>; the second group comprises the testcases from the <code class="samp docutils literal notranslate"><em><span class="pre">t1</span></em></code> + 1 to the <code class="samp docutils literal notranslate"><em><span class="pre">t1</span></em></code> + <code class="samp docutils literal notranslate"><em><span class="pre">t2</span></em></code> and has multiplier <code class="samp docutils literal notranslate"><em><span class="pre">m2</span></em></code>; and so on.</p>
<p>In the second case (second element is always a string), the string is interpreted as a regular expression, and the group comprises all testcases whose codename satisfies it.</p>
</div>
<div class="section" id="groupmul">
<h3>GroupMul<a class="headerlink" href="#groupmul" title="Permalink to this headline">¶</a></h3>
<p>GroupMul is almost the same as GroupMin; the only difference is that instead of taking the minimum outcome among the testcases in the group, it takes the product of all outcomes. It has the same behavior as GroupMin when all outcomes are either 0.0 or 1.0.</p>
</div>
<div class="section" id="groupthreshold">
<h3>GroupThreshold<a class="headerlink" href="#groupthreshold" title="Permalink to this headline">¶</a></h3>
<p>GroupThreshold thinks of the outcomes not as a measure of success, but as an amount of resources used by the submission to solve the testcase. The testcase is then successfully solved if the outcome is between 0.0 (excluded, as 0.0 is a special value used by many task types, for example when the contestant solution times out) and a certain number, the threshold, specified separately for each group.</p>
<p>The parameter string is of the form <code class="samp docutils literal notranslate"><span class="pre">[[</span><em><span class="pre">m1</span></em><span class="pre">,</span> <em><span class="pre">t1</span></em><span class="pre">,</span> <em><span class="pre">T1</span></em><span class="pre">],</span> <span class="pre">[</span><em><span class="pre">m2</span></em><span class="pre">,</span> <em><span class="pre">t2</span></em><span class="pre">,</span> <em><span class="pre">T2</span></em><span class="pre">],</span> <span class="pre">...]</span></code> where the additional parameter <code class="samp docutils literal notranslate"><em><span class="pre">T</span></em></code> for each group is the threshold.</p>
<p>The task needs to be crafted in such a way that the meaning of the outcome is appropriate for this score type.</p>
<p>For Batch tasks, this means that the tasks creates the outcome through a comparator program. Using diff does not make sense given that its outcomes can only be 0.0 or 1.0.</p>
</div>
</div>
<div class="section" id="custom-score-types">
<h2>Custom score types<a class="headerlink" href="#custom-score-types" title="Permalink to this headline">¶</a></h2>
<p>Additional score types can be defined if necessary. This works in the same way <a class="reference internal" href="index.html#tasktypes-custom"><span class="std std-ref">as with task types</span></a>: the classes need to extend <code class="xref py py-class docutils literal notranslate"><span class="pre">cms.grading.scoretypes.ScoreType</span></code> and the entry point group is called <cite>cms.grading.scoretypes</cite>.</p>
</div>
</div>
<span id="document-Task versioning"></span><div class="section" id="task-versioning">
<h1>Task versioning<a class="headerlink" href="#task-versioning" title="Permalink to this headline">¶</a></h1>
<div class="section" id="introduction">
<h2>Introduction<a class="headerlink" href="#introduction" title="Permalink to this headline">¶</a></h2>
<p>Task versioning allows admins to store several sets of parameters for each task at the same time, to decide which are graded and among these the one that is shown to the contestants. This is useful before the contest, to test different possibilities, but especially during the contest to investigate the impact of an error in the task preparation.</p>
<p>For example, it is quite common to realize that one input file is wrong. With task versioning, admins can clone the original dataset (the set of parameters describing the behavior of the task), change the wrong input file with another one, or delete it, launch the evaluation on the new dataset, see which contestants have been affected by the problem, and finally swap the two datasets to make the new one live and visible by the contestants.</p>
<p>The advantages over the situation without task versioning are several:</p>
<ul class="simple">
<li>there is no need to take down scores during the re-evaluation with the new input;</li>
<li>it is possible to make sure that the new input works well without showing anything to the contestants;</li>
<li>if the problem affects just a few contestants, it is possible to notify just them, and the others will be completely unaffected.</li>
</ul>
</div>
<div class="section" id="datasets">
<h2>Datasets<a class="headerlink" href="#datasets" title="Permalink to this headline">¶</a></h2>
<p>A dataset is a version of the sets of parameters of a task that can be changed and tested in background. These parameters are:</p>
<ul class="simple">
<li>time and memory limits;</li>
<li>input and output files;</li>
<li>libraries and graders;</li>
<li>task type and score type.</li>
</ul>
<p>Datasets can be viewed and edited in the task page. They can be created from scratch or cloned from existing ones. Of course, during a contest cloning the live dataset is the most used way of creating a new one.</p>
<p>Submissions are evaluated as they arrive against the live dataset and all other datasets with background judging enabled, or on demand when the admins require it.</p>
<p>Each task has exactly one live dataset, whose evaluations and scores are shown to the contestants. To change the live dataset, just click on “Make live” on the desired dataset. Admins will then be prompted with a summary of what changed between the new dataset and the previously active, and can decide to cancel or go ahead, possibly notifying the contestants with a message.</p>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">Remember that the summary looks at the scores currently stored for each submission. This means that if you cloned a dataset and changed an input, the scores will still be the old ones: you need to launch a recompilation, reevaluation, or rescoring, depending on what you changed, before seeing the new scores.</p>
</div>
<p>After switching live dataset, scores will be resent to RankingWebServer automatically.</p>
</div>
</div>
<span id="document-External contest formats"></span><div class="section" id="external-contest-formats">
<h1>External contest formats<a class="headerlink" href="#external-contest-formats" title="Permalink to this headline">¶</a></h1>
<p>There are two different sets of needs that external contest formats strive to satisfy.</p>
<ul class="simple">
<li>The first is that of contest admins, that for several reasons (storage of old contests, backup, distribution of data) want to export the contest original data (tasks, contestants, …) together with all data generated during the contest (from the contestants, submissions, user tests, … and from the system, evaluations, scores, …). Once a contest has been exported in this format, CMS must be able to reimport it in such a way that the new instance is indistinguishable from the original.</li>
<li>The second is that of contest creators, that want an environment that helps them design tasks, testcases, and insert the contest data (contestant names and so on). The format needs to be easy to write, understand and modify, and should provide tools to help developing and testing the tasks (automatic generation of testcases, testing of solutions, …). CMS must be able to import it as a new contest, but also to import it over an already created contest (after updating some data).</li>
</ul>
<p>CMS provides an exporter <code class="file docutils literal notranslate"><span class="pre">cmsDumpExporter</span></code> and an importer <code class="file docutils literal notranslate"><span class="pre">cmsDumpImporter</span></code> working with a format suitable for the first set of needs. This format comprises a dump of all serializable data regarding the contest in a JSON file, together with the files needed by the contest (testcases, statements, submissions, user tests, …). The exporter and importer understand also compressed versions of this format (i.e., in a zip or tar file). For more information run</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>cmsDumpExporter -h
cmsDumpImporter -h
</pre></div>
</div>
<p>As for the second set of needs, the philosophy is that CMS should not force upon contest creators a particular environment to write contests and tasks. Therefore, CMS provides general-purpose commands, <code class="file docutils literal notranslate"><span class="pre">cmsAddUser</span></code>, <code class="file docutils literal notranslate"><span class="pre">cmsAddTask</span></code> and <code class="file docutils literal notranslate"><span class="pre">cmsAddContest</span></code>. These programs have no knowledge of any specific on-disk format, so they must be complemented with a set of “loaders”, which actually interpret your files and directories. You can tell the importer or the reimported wich loader to use with the <code class="docutils literal notranslate"><span class="pre">-L</span></code> flag, or just rely and their autodetection capabilities. Running with <code class="docutils literal notranslate"><span class="pre">-h</span></code> flag will list the available loaders.</p>
<p>At the moment, CMS comes with two loaders pre-installed:</p>
<ul class="simple">
<li><code class="file docutils literal notranslate"><span class="pre">italy_yaml</span></code>, for tasks/users stored in the “Italian Olympiad” format.</li>
<li><code class="file docutils literal notranslate"><span class="pre">polygon_xml</span></code>, for tasks made with <a class="reference external" href="https://polygon.codeforces.com/">Polygon</a>.</li>
</ul>
<p>The first one is not particularly suited for general use (see below for more details), so, if you don’t want to migrate to one of the aforementioned formats then we encourage you to <strong>write a loader</strong> for your favorite format and then get in touch with CMS authors to have it accepted in CMS. See the file <a class="reference external" href="https://github.com/cms-dev/cms/blob/v1.5.dev0/cmscontrib/loaders/base_loader.py">cmscontrib/loaders/base_loader.py</a> for some hints.</p>
<div class="section" id="italian-import-format">
<h2>Italian import format<a class="headerlink" href="#italian-import-format" title="Permalink to this headline">¶</a></h2>
<p>You can follow this description looking at <a class="reference external" href="https://github.com/cms-dev/con_test">this example</a>. A contest is represented in one directory, containing:</p>
<ul class="simple">
<li>a YAML file named <code class="file docutils literal notranslate"><span class="pre">contest.yaml</span></code>, that describes the general contest properties;</li>
<li>for each task <code class="samp docutils literal notranslate"><em><span class="pre">task_name</span></em></code>, a directory <code class="file docutils literal notranslate"><em><span class="pre">task_name</span></em></code> that contains the description of the task and all the files needed to build the statement of the problem, the input and output cases, the reference solution and (when used) the solution checker.</li>
</ul>
<p>The exact structure of these files and directories is detailed below. Note that this loader is not particularly reliable and providing confusing input to it may lead to create inconsistent or strange data on the database. For confusing input we mean parameters and/or files from which it can infer no or multiple task types or score types.</p>
<p>As the name suggest, this format was born among the Italian trainers group, thus many of the keywords detailed below used to be in Italian. Now they have been translated to English, but Italian keys are still recognized for backward compatibility and are detailed below. Please note that, although so far this is the only format natively supported by CMS, it is far from ideal: in particular, it has grown in a rather untidy manner in the last few years (CMS authors are planning to develop a new, more general and more organic, format, but unfortunately it doesn’t exist yet).</p>
<p>For the reasons above, instead of converting your tasks to the Italian format for importing into CMS, it is suggested to write a loader for the format you already have. Please get in touch with CMS authors to have support.</p>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p class="last">The authors offer no guarantee for future compatibility for this format. Again, if you use it, you do so at your own risk!</p>
</div>
<div class="section" id="general-contest-description">
<h3>General contest description<a class="headerlink" href="#general-contest-description" title="Permalink to this headline">¶</a></h3>
<p>The <code class="file docutils literal notranslate"><span class="pre">contest.yaml</span></code> file is a plain YAML file, with at least the following keys.</p>
<ul class="simple">
<li><code class="docutils literal notranslate"><span class="pre">name</span></code> (string; also accepted: <code class="docutils literal notranslate"><span class="pre">nome_breve</span></code>): the contest’s short name, used for internal reference (and exposed in the URLs); it has to match the name of the directory that serves as contest root.</li>
<li><code class="docutils literal notranslate"><span class="pre">description</span></code> (string; also accepted: <code class="docutils literal notranslate"><span class="pre">nome</span></code>): the contest’s name (description), shown to contestants in the web interface.</li>
<li><code class="docutils literal notranslate"><span class="pre">tasks</span></code> (list of strings; also accepted: <code class="docutils literal notranslate"><span class="pre">problemi</span></code>): a list of the tasks belonging to this contest; for each of these strings, say <code class="samp docutils literal notranslate"><em><span class="pre">task_name</span></em></code>, there must be a directory called <code class="file docutils literal notranslate"><em><span class="pre">task_name</span></em></code> in the contest directory, with content as described <a class="reference internal" href="#externalcontestformats-task-directory"><span class="std std-ref">below</span></a>; the order in this list will be the order of the tasks in the web interface.</li>
<li><code class="docutils literal notranslate"><span class="pre">users</span></code> (list of associative arrays; also accepted: <code class="docutils literal notranslate"><span class="pre">utenti</span></code>): each of the elements of the list describes one user of the contest; the exact structure of the record is described <a class="reference internal" href="#externalcontestformats-user-description"><span class="std std-ref">below</span></a>.</li>
<li><code class="docutils literal notranslate"><span class="pre">token_mode</span></code>: the token mode for the contest, as in <a class="reference internal" href="index.html#configuringacontest-tokens"><span class="std std-ref">Tokens rules</span></a>; it can be <code class="docutils literal notranslate"><span class="pre">disabled</span></code>, <code class="docutils literal notranslate"><span class="pre">infinite</span></code> or <code class="docutils literal notranslate"><span class="pre">finite</span></code>; if this is not specified, the loader will try to infer it from the remaining token parameters (in order to retain compatibility with the past), but you are not advised to rely on this behavior.</li>
</ul>
<p>The following are optional keys.</p>
<ul class="simple">
<li><code class="docutils literal notranslate"><span class="pre">start</span></code> (integer; also accepted: <code class="docutils literal notranslate"><span class="pre">inizio</span></code>): the UNIX timestamp of the beginning of the contest (copied in the <code class="docutils literal notranslate"><span class="pre">start</span></code> field); defaults to zero, meaning that contest times haven’t yet been decided.</li>
<li><code class="docutils literal notranslate"><span class="pre">stop</span></code> (integer; also accepted: <code class="docutils literal notranslate"><span class="pre">fine</span></code>): the UNIX timestamp of the end of the contest (copied in the <code class="docutils literal notranslate"><span class="pre">stop</span></code> field); defaults to zero, meaning that contest times haven’t yet been decided.</li>
<li><code class="docutils literal notranslate"><span class="pre">timezone</span></code> (string): the timezone for the contest (e.g., “Europe/Rome”).</li>
<li><code class="docutils literal notranslate"><span class="pre">per_user_time</span></code> (integer): if set, the contest will be USACO-like (as explained in <a class="reference internal" href="index.html#configuringacontest-usaco-like-contests"><span class="std std-ref">USACO-like contests</span></a>); if unset, the contest will be traditional (not USACO-like).</li>
<li><code class="docutils literal notranslate"><span class="pre">token_*</span></code>: additional token parameters for the contest, see <a class="reference internal" href="index.html#configuringacontest-tokens"><span class="std std-ref">Tokens rules</span></a> (the names of the parameters are the same as the internal names described there).</li>
<li><code class="docutils literal notranslate"><span class="pre">max_*_number</span></code> and <code class="docutils literal notranslate"><span class="pre">min_*_interval</span></code> (integers): limitations for the whole contest, see <a class="reference internal" href="index.html#configuringacontest-limitations"><span class="std std-ref">Limitations</span></a> (the names of the parameters are the same as the internal names described there); by default they’re all unset.</li>
</ul>
</div>
<div class="section" id="user-description">
<span id="externalcontestformats-user-description"></span><h3>User description<a class="headerlink" href="#user-description" title="Permalink to this headline">¶</a></h3>
<p>Each contest user (contestant) is described in one element of the <code class="docutils literal notranslate"><span class="pre">utenti</span></code> key in the <code class="file docutils literal notranslate"><span class="pre">contest.yaml</span></code> file. Each record has to contains the following keys.</p>
<ul class="simple">
<li><code class="docutils literal notranslate"><span class="pre">username</span></code> (string): obviously, the username.</li>
<li><code class="docutils literal notranslate"><span class="pre">password</span></code> (string): obviously as before, the user’s password.</li>
</ul>
<p>The following are optional keys.</p>
<ul class="simple">
<li><code class="docutils literal notranslate"><span class="pre">first_name</span></code> (string; also accepted: <code class="docutils literal notranslate"><span class="pre">nome</span></code>): the user real first name; defaults to the empty string.</li>
<li><code class="docutils literal notranslate"><span class="pre">last_name</span></code> (string; also accepted: <code class="docutils literal notranslate"><span class="pre">cognome</span></code>): the user real last name; defaults to the value of <code class="docutils literal notranslate"><span class="pre">username</span></code>.</li>
<li><code class="docutils literal notranslate"><span class="pre">ip</span></code> (string): the IP address or subnet from which incoming connections for this user are accepted, see <a class="reference internal" href="index.html#configuringacontest-login"><span class="std std-ref">User login</span></a>.</li>
<li><code class="docutils literal notranslate"><span class="pre">hidden</span></code> (boolean; also accepted: <code class="docutils literal notranslate"><span class="pre">fake</span></code>): when set to true set the <code class="docutils literal notranslate"><span class="pre">hidden</span></code> flag in the user, see <a class="reference internal" href="index.html#configuringacontest-login"><span class="std std-ref">User login</span></a>; defaults to false (the case-sensitive <em>string</em> <code class="docutils literal notranslate"><span class="pre">True</span></code> is also accepted).</li>
</ul>
</div>
<div class="section" id="task-directory">
<span id="externalcontestformats-task-directory"></span><h3>Task directory<a class="headerlink" href="#task-directory" title="Permalink to this headline">¶</a></h3>
<p>The content of the task directory is used both to retrieve the task data and to infer the type of the task.</p>
<p>These are the required files.</p>
<ul class="simple">
<li><code class="file docutils literal notranslate"><span class="pre">task.yaml</span></code>: this file contains the name of the task and describes some of its properties; its content is detailed <a class="reference internal" href="#externalcontestformats-task-description"><span class="std std-ref">below</span></a>; in order to retain backward compatibility, this file can also be provided in the file <code class="file docutils literal notranslate"><em><span class="pre">task_name.yaml</span></em></code> in the root directory of the <em>contest</em>.</li>
<li><code class="file docutils literal notranslate"><span class="pre">statement/statement.pdf</span></code> (also accepted: <code class="file docutils literal notranslate"><span class="pre">testo/testo.pdf</span></code>): the main statement of the problem. It is not yet possible to import several statement associated to different languages: this (only) statement will be imported according to the language specified under the key <code class="docutils literal notranslate"><span class="pre">primary_language</span></code>.</li>
<li><code class="file docutils literal notranslate"><span class="pre">input/input</span><em><span class="pre">%d</span></em><span class="pre">.txt</span></code> and <code class="file docutils literal notranslate"><span class="pre">output/output</span><em><span class="pre">%d</span></em><span class="pre">.txt</span></code> for all integers <code class="samp docutils literal notranslate"><em><span class="pre">%d</span></em></code> between 0 (included) and <code class="docutils literal notranslate"><span class="pre">n_input</span></code> (excluded): these are of course the input and reference output files.</li>
</ul>
<p>The following are optional files, that must be present for certain task types or score types.</p>
<ul class="simple">
<li><code class="file docutils literal notranslate"><span class="pre">gen/GEN</span></code>: in the Italian environment, this file describes the parameters for the input generator: each line not composed entirely by white spaces or comments (comments start with <code class="docutils literal notranslate"><span class="pre">#</span></code> and end with the end of the line) represents an input file. Here, it is used, in case it contains specially formatted comments, to signal that the score type is <a class="reference internal" href="index.html#scoretypes-groupmin"><span class="std std-ref">GroupMin</span></a>. If a line contains only a comment of the form <code class="samp docutils literal notranslate"><span class="pre">#</span> <span class="pre">ST:</span> <em><span class="pre">score</span></em></code> then it marks the beginning of a new group assigning at most <code class="samp docutils literal notranslate"><em><span class="pre">score</span></em></code> points, containing all subsequent testcases until the next special comment. If the file does not exists, or does not contain any special comments, the task is given the <a class="reference internal" href="index.html#scoretypes-sum"><span class="std std-ref">Sum</span></a> score type.</li>
<li><code class="file docutils literal notranslate"><span class="pre">sol/grader.</span><em><span class="pre">%l</span></em></code> (where <code class="samp docutils literal notranslate"><em><span class="pre">%l</span></em></code> here and after means a supported language extension): for tasks of type <a class="reference internal" href="index.html#tasktypes-batch"><span class="std std-ref">Batch</span></a>, it is the piece of code that gets compiled together with the submitted solution, and usually takes care of reading the input and writing the output. If one grader is present, the graders for all supported languages must be provided.</li>
<li><code class="file docutils literal notranslate"><span class="pre">sol/*.h</span></code> and <code class="file docutils literal notranslate"><span class="pre">sol/*lib.pas</span></code>: if a grader is present, all other files in the <code class="file docutils literal notranslate"><span class="pre">sol</span></code> directory that end with <code class="docutils literal notranslate"><span class="pre">.h</span></code> or <code class="docutils literal notranslate"><span class="pre">lib.pas</span></code> are treated as auxiliary files needed by the compilation of the grader with the submitted solution.</li>
<li><code class="file docutils literal notranslate"><span class="pre">check/checker</span></code> (also accepted: <code class="file docutils literal notranslate"><span class="pre">cor/correttore</span></code>): for tasks of types <a class="reference internal" href="index.html#tasktypes-batch"><span class="std std-ref">Batch</span></a> or <a class="reference internal" href="index.html#tasktypes-outputonly"><span class="std std-ref">OutputOnly</span></a>, if this file is present, it must be the executable that examines the input and both the correct and the contestant’s output files and assigns the outcome. It must be a statically linked executable (for example, if compiled from a C or C++ source, the <code class="samp docutils literal notranslate"><span class="pre">-static</span></code> option must be used) because otherwise the sandbox will prevent it from accessing its dependencies. It is going to be executed on the workers, so it must be compiled for their architecture. If instead the file is not present, a simple diff is used to compare the correct and the contestant’s output files.</li>
<li><code class="file docutils literal notranslate"><span class="pre">check/manager</span></code>: (also accepted: <code class="file docutils literal notranslate"><span class="pre">cor/manager</span></code>) for tasks of type <a class="reference internal" href="index.html#tasktypes-communication"><span class="std std-ref">Communication</span></a>, this executable is the program that reads the input and communicates with the user solution.</li>
<li><code class="file docutils literal notranslate"><span class="pre">sol/stub.%l</span></code>: for tasks of type <a class="reference internal" href="index.html#tasktypes-communication"><span class="std std-ref">Communication</span></a>, this is the piece of code that is compiled together with the user submitted code, and is usually used to manage the communication with <code class="file docutils literal notranslate"><span class="pre">manager</span></code>. Again, all supported languages must be present.</li>
<li><code class="file docutils literal notranslate"><span class="pre">att/*</span></code>: each file in this folder is added as an attachment to the task, named as the file’s filename.</li>
</ul>
</div>
<div class="section" id="task-description">
<span id="externalcontestformats-task-description"></span><h3>Task description<a class="headerlink" href="#task-description" title="Permalink to this headline">¶</a></h3>
<p>The task YAML files require the following keys.</p>
<ul class="simple">
<li><code class="docutils literal notranslate"><span class="pre">name</span></code> (string; also accepted: <code class="docutils literal notranslate"><span class="pre">nome_breve</span></code>): the name used to reference internally to this task; it is exposed in the URLs.</li>
<li><code class="docutils literal notranslate"><span class="pre">title</span></code> (string; also accepted: <code class="docutils literal notranslate"><span class="pre">nome</span></code>): the long name (title) used in the web interface.</li>
<li><code class="docutils literal notranslate"><span class="pre">n_input</span></code> (integer): number of test cases to be evaluated for this task; the actual test cases are retrieved from the <a class="reference internal" href="#externalcontestformats-task-directory"><span class="std std-ref">task directory</span></a>.</li>
<li><code class="docutils literal notranslate"><span class="pre">score_mode</span></code>: the score mode for the task, as in <a class="reference internal" href="index.html#configuringacontest-score"><span class="std std-ref">Computation of the score</span></a>; it can be <code class="docutils literal notranslate"><span class="pre">max_tokened_last</span></code>, <code class="docutils literal notranslate"><span class="pre">max</span></code>, or <code class="docutils literal notranslate"><span class="pre">max_subtask</span></code>.</li>
<li><code class="docutils literal notranslate"><span class="pre">token_mode</span></code>: the token mode for the task, as in <a class="reference internal" href="index.html#configuringacontest-tokens"><span class="std std-ref">Tokens rules</span></a>; it can be <code class="docutils literal notranslate"><span class="pre">disabled</span></code>, <code class="docutils literal notranslate"><span class="pre">infinite</span></code> or <code class="docutils literal notranslate"><span class="pre">finite</span></code>; if this is not specified, the loader will try to infer it from the remaining token parameters (in order to retain compatibility with the past), but you are not advised to relay on this behavior.</li>
</ul>
<p>The following are optional keys.</p>
<ul class="simple">
<li><code class="docutils literal notranslate"><span class="pre">time_limit</span></code> (float; also accepted: <code class="docutils literal notranslate"><span class="pre">timeout</span></code>): the timeout limit for this task in seconds; defaults to no limitations.</li>
<li><code class="docutils literal notranslate"><span class="pre">memory_limit</span></code> (integer; also accepted: <code class="docutils literal notranslate"><span class="pre">memlimit</span></code>): the memory limit for this task in mibibytes; defaults to no limitations.</li>
<li><code class="docutils literal notranslate"><span class="pre">public_testcases</span></code> (string; also accepted: <code class="docutils literal notranslate"><span class="pre">risultati</span></code>): a comma-separated list of test cases (identified by their numbers, starting from 0) that are marked as public, hence their results are available to contestants even without using tokens. If the given string is equal to <code class="docutils literal notranslate"><span class="pre">all</span></code>, then the importer will mark all testcases as public.</li>
<li><code class="docutils literal notranslate"><span class="pre">token_*</span></code>: additional token parameters for the task, see <a class="reference internal" href="index.html#configuringacontest-tokens"><span class="std std-ref">Tokens rules</span></a> (the names of the parameters are the same as the internal names described there).</li>
<li><code class="docutils literal notranslate"><span class="pre">max_*_number</span></code> and <code class="docutils literal notranslate"><span class="pre">min_*_interval</span></code> (integers): limitations for the task, see <a class="reference internal" href="index.html#configuringacontest-limitations"><span class="std std-ref">Limitations</span></a> (the names of the parameters are the same as the internal names described there); by default they’re all unset.</li>
<li><code class="docutils literal notranslate"><span class="pre">output_only</span></code> (boolean): if set to True, the task is created with the <a class="reference internal" href="index.html#tasktypes-outputonly"><span class="std std-ref">OutputOnly</span></a> type; defaults to False.</li>
</ul>
<p>The following are optional keys that must be present for some task type or score type.</p>
<ul class="simple">
<li><code class="docutils literal notranslate"><span class="pre">total_value</span></code> (float): for tasks using the <a class="reference internal" href="index.html#scoretypes-sum"><span class="std std-ref">Sum</span></a> score type, this is the maximum score for the task and defaults to 100.0; for other score types, the maximum score is computed from the <a class="reference internal" href="#externalcontestformats-task-directory"><span class="std std-ref">task directory</span></a>.</li>
<li><code class="docutils literal notranslate"><span class="pre">infile</span></code> and <code class="docutils literal notranslate"><span class="pre">outfile</span></code> (strings): for <a class="reference internal" href="index.html#tasktypes-batch"><span class="std std-ref">Batch</span></a> tasks, these are the file names for the input and output files; default to <code class="file docutils literal notranslate"><span class="pre">input.txt</span></code> and <code class="file docutils literal notranslate"><span class="pre">output.txt</span></code>; if left empty, <code class="file docutils literal notranslate"><span class="pre">stdin</span></code> and <code class="file docutils literal notranslate"><span class="pre">stdout</span></code> are used.</li>
<li><code class="docutils literal notranslate"><span class="pre">primary_language</span></code> (string): the statement will be imported with this language code; defaults to <code class="docutils literal notranslate"><span class="pre">it</span></code> (Italian), in order to ensure backward compatibility.</li>
</ul>
</div>
</div>
<div class="section" id="polygon-format">
<h2>Polygon format<a class="headerlink" href="#polygon-format" title="Permalink to this headline">¶</a></h2>
<p><a class="reference external" href="https://polygon.codeforces.com">Polygon</a> is a popular platform for the creation of tasks, and a task format, used among others by Codeforces.</p>
<p>Since Polygon doesn’t support CMS directly, some task parameters cannot be set using the standard Polygon configuration. The importer reads from an optional file <code class="file docutils literal notranslate"><span class="pre">cms_conf.py</span></code> additional configuration specifics to CMS. Additionally, user can add file named contestants.txt to allow importing some set of users.</p>
<p>By default, all tasks are batch files, with custom checker and score type is Sum. Loaders assumes that checker is check.cpp and written with usage of testlib.h. It provides customized version of testlib.h which allows using Polygon checkers with CMS. Checkers will be compiled during importing the contest. This is important in case the architecture where the loading happens is different from the architecture of the workers.</p>
<p>Polygon (by now) doesn’t allow custom contest-wide files, so general contest options should be hard-coded in the loader.</p>
</div>
</div>
<span id="document-RankingWebServer"></span><div class="section" id="rankingwebserver">
<h1>RankingWebServer<a class="headerlink" href="#rankingwebserver" title="Permalink to this headline">¶</a></h1>
<div class="section" id="description">
<h2>Description<a class="headerlink" href="#description" title="Permalink to this headline">¶</a></h2>
<p>The <strong>RankingWebServer</strong> (RWS for short) is the web server used to show a live scoreboard to the public.</p>
<p>RWS is designed to be completely separated from the rest of CMS: it has its own configuration file, it doesn’t use the PostgreSQL database to store its data and it doesn’t communicate with other services using the internal RPC protocol (its code is also in a different package: <code class="docutils literal notranslate"><span class="pre">cmsranking</span></code> instead of <code class="docutils literal notranslate"><span class="pre">cms</span></code>). This has been done to allow contest administrators to run RWS in a different location (on a different network) than the core of CMS, if they don’t want to expose a public access to their core network on the internet (for security reasons) or if the on-site internet connection isn’t good enough to serve a public website.</p>
<p>To start RWS you have to execute <code class="docutils literal notranslate"><span class="pre">cmsRankingWebServer</span></code>.</p>
<div class="section" id="configuring-it">
<h3>Configuring it<a class="headerlink" href="#configuring-it" title="Permalink to this headline">¶</a></h3>
<p>The configuration file is named <code class="file docutils literal notranslate"><span class="pre">cms.ranking.conf</span></code> and RWS will search for it in <code class="file docutils literal notranslate"><span class="pre">/usr/local/etc</span></code> and in <code class="file docutils literal notranslate"><span class="pre">/etc</span></code> (in this order!). In case it’s not found in any of these, RWS will use a hard-coded default configuration that can be found in <a class="reference external" href="https://github.com/cms-dev/cms/blob/v1.5.dev0/cmsranking/Config.py">cmsranking/Config.py</a>. If RWS is not installed then the <a class="reference external" href="https://github.com/cms-dev/cms/tree/v1.5.dev0/config">config</a> directory will also be checked for configuration files (note that for this to work your working directory needs to be root of the repository). In any case, as soon as you start it, RWS will tell you which configuration file it’s using.</p>
<p>The configuration file is a JSON object. The most important parameters are:</p>
<ul>
<li><p class="first"><code class="docutils literal notranslate"><span class="pre">bind_address</span></code></p>
<p>It specifies the address this server will listen on. It can be either an IP address or a hostname (in the latter case the server will listen on all IP addresses associated with that name). Leave it blank or set it to <code class="docutils literal notranslate"><span class="pre">null</span></code> to listen on all available interfaces.</p>
</li>
<li><p class="first"><code class="docutils literal notranslate"><span class="pre">http_port</span></code></p>
<p>It specifies which port to bind the HTTP server to. If set to <code class="docutils literal notranslate"><span class="pre">null</span></code> it will be disabled. We suggest to use a high port number (like 8080, or the default 8890) to avoid the need to start RWS as root, and then use a reverse proxy to map port 80 to it (see <a class="reference internal" href="#rankingwebserver-using-a-proxy"><span class="std std-ref">Using a proxy</span></a> for additional information).</p>
</li>
<li><p class="first"><code class="docutils literal notranslate"><span class="pre">https_port</span></code></p>
<p>It specifies which port to bind the HTTPS server to. If set to <code class="docutils literal notranslate"><span class="pre">null</span></code> it will be disabled, otherwise you need to set <code class="docutils literal notranslate"><span class="pre">https_certfile</span></code> and <code class="docutils literal notranslate"><span class="pre">https_keyfile</span></code> too. See <a class="reference internal" href="#rankingwebserver-securing-the-connection-between-ss-and-rws"><span class="std std-ref">Securing the connection between PS and RWS</span></a> for additional information.</p>
</li>
<li><p class="first"><code class="docutils literal notranslate"><span class="pre">username</span></code> and <code class="docutils literal notranslate"><span class="pre">password</span></code></p>
<p>They specify the credentials needed to alter the data of RWS. We suggest to set them to long random strings, for maximum security, since you won’t need to remember them. <code class="docutils literal notranslate"><span class="pre">username</span></code> cannot contain a colon.</p>
<div class="admonition warning">
<p class="first admonition-title">Warning</p>
<p class="last">Remember to change the <code class="docutils literal notranslate"><span class="pre">username</span></code> and <code class="docutils literal notranslate"><span class="pre">password</span></code> every time you set up a RWS. Keeping the default ones will leave your scoreboard open to illegitimate access.</p>
</div>
</li>
</ul>
<p>To connect the rest of CMS to your new RWS you need to add its connection parameters to the configuration file of CMS (i.e. <code class="file docutils literal notranslate"><span class="pre">cms.conf</span></code>). Note that you can connect CMS to multiple RWSs, each on a different server and/or port. The parameter you need to change is <code class="docutils literal notranslate"><span class="pre">rankings</span></code>, a list of URLs in the form:</p>
<div class="highlight-default notranslate"><div class="highlight"><pre><span></span><span class="o">&lt;</span><span class="n">scheme</span><span class="o">&gt;</span><span class="p">:</span><span class="o">//&lt;</span><span class="n">username</span><span class="o">&gt;</span><span class="p">:</span><span class="o">&lt;</span><span class="n">password</span><span class="o">&gt;@&lt;</span><span class="n">hostname</span><span class="o">&gt;</span><span class="p">:</span><span class="o">&lt;</span><span class="n">port</span><span class="o">&gt;/&lt;</span><span class="n">prefix</span><span class="o">&gt;</span>
</pre></div>
</div>
<p>where <code class="docutils literal notranslate"><span class="pre">scheme</span></code> can be either <code class="docutils literal notranslate"><span class="pre">http</span></code> or <code class="docutils literal notranslate"><span class="pre">https</span></code>, <code class="docutils literal notranslate"><span class="pre">username</span></code>, <code class="docutils literal notranslate"><span class="pre">password</span></code> and <code class="docutils literal notranslate"><span class="pre">port</span></code> are the values specified in the configuration file of the RWS and <code class="docutils literal notranslate"><span class="pre">prefix</span></code> is explained in <a class="reference internal" href="#rankingwebserver-using-a-proxy"><span class="std std-ref">Using a proxy</span></a> (it will generally be blank, otherwise it needs to end with a slash). If any of your RWSs uses the HTTPS protocol you also need to specify the <code class="docutils literal notranslate"><span class="pre">https_certfile</span></code> configuration parameter. More details on this in <a class="reference internal" href="#rankingwebserver-securing-the-connection-between-ss-and-rws"><span class="std std-ref">Securing the connection between PS and RWS</span></a>.</p>
<p>You also need to make sure that RWS is able to keep enough simultaneously active connections by checking that the maximum number of open file descriptors is larger than the expected number of clients. You can see the current value with <code class="docutils literal notranslate"><span class="pre">ulimit</span> <span class="pre">-Sn</span></code> (or <code class="docutils literal notranslate"><span class="pre">-Sa</span></code> to see all limitations) and change it with <code class="docutils literal notranslate"><span class="pre">ulimit</span> <span class="pre">-Sn</span> <span class="pre">&lt;value&gt;</span></code>. This value will be reset when you open a new shell, so remember to run the command again. Note that there may be a hard limit that you cannot overcome (use <code class="docutils literal notranslate"><span class="pre">-H</span></code> instead of <code class="docutils literal notranslate"><span class="pre">-S</span></code> to see it). If that’s still too low you can start multiple RWSs and use a proxy to distribute clients among them (see <a class="reference internal" href="#rankingwebserver-using-a-proxy"><span class="std std-ref">Using a proxy</span></a>).</p>
</div>
</div>
<div class="section" id="managing-data">
<h2>Managing data<a class="headerlink" href="#managing-data" title="Permalink to this headline">¶</a></h2>
<p>RWS doesn’t use the PostgreSQL database. Instead, it stores its data in <code class="file docutils literal notranslate"><span class="pre">/var/local/lib/cms/ranking</span></code> (or whatever directory is given as <code class="docutils literal notranslate"><span class="pre">lib_dir</span></code> in the configuration file) as a collection of JSON files. Thus, if you want to backup the RWS data, just make a copy of that directory. RWS modifies this data in response to specific (authenticated) HTTP requests it receives.</p>
<p>The intended way to get data to RWS is to have the rest of CMS send it. The service responsible for that is ProxyService (PS for short). When PS is started for a certain contest, it will send the data for that contest to all RWSs it knows about (i.e. those in its configuration). This data includes the contest itself (its name, its begin and end times, etc.), its tasks, its users and teams, and the submissions received so far. Then it will continue to send new submissions as soon as they are scored and it will update them as needed (for example when a user uses a token). Note that hidden users (and their submissions) will not be sent to RWS.</p>
<p>There are also other ways to insert data into RWS: send custom HTTP requests or directly write JSON files. For the former, the script <cite>cmsRWSHelper</cite> can be used to handle the low level communication.</p>
<div class="section" id="logo-flags-and-faces">
<h3>Logo, flags and faces<a class="headerlink" href="#logo-flags-and-faces" title="Permalink to this headline">¶</a></h3>
<p>RWS can also display a custom global logo, a flag for each team and a photo (“face”) for each user. The only way to add these is to put them directly in the data directory of RWS:</p>
<ul class="simple">
<li>the logo has to be saved right in the data directory, named “logo” with an appropriate extension (e.g. <code class="file docutils literal notranslate"><span class="pre">logo.png</span></code>), with a recommended resolution of 200x160;</li>
<li>the flag for a team has to be saved in the “flags” subdirectory, named as the team’s name with an appropriate extension (e.g. <code class="file docutils literal notranslate"><span class="pre">ITA.png</span></code>);</li>
<li>the face for a user has to be saved in the “faces” subdirectory, named as the user’s username with an appropriate extension (e.g. <code class="file docutils literal notranslate"><span class="pre">ITA1.png</span></code>).</li>
</ul>
<p>We support the following extensions: .png, .jpg, .gif and .bmp.</p>
</div>
<div class="section" id="removing-data">
<span id="rankingwebserver-removing-data"></span><h3>Removing data<a class="headerlink" href="#removing-data" title="Permalink to this headline">¶</a></h3>
<p>PS is only able to create or update data on RWS, but not to delete it. This means that, for example, when a user or a task is removed from CMS it will continue to be shown on RWS. To fix this you will have to intervene manually. The <code class="docutils literal notranslate"><span class="pre">cmsRWSHelper</span></code> script is designed to make this operation straightforward. For example, calling <code class="samp docutils literal notranslate"><span class="pre">cmsRWSHelper</span> <span class="pre">delete</span> <span class="pre">user</span> <em><span class="pre">username</span></em></code> will cause the user <em>username</em> to be removed from all the RWSs that are specified in <code class="file docutils literal notranslate"><span class="pre">cms.conf</span></code>. See <code class="docutils literal notranslate"><span class="pre">cmsRWSHelper</span> <span class="pre">--help</span></code> and <code class="samp docutils literal notranslate"><span class="pre">cmsRWSHelper</span> <em><span class="pre">action</span></em> <span class="pre">--help</span></code> for more usage details.</p>
<p>In case using <code class="docutils literal notranslate"><span class="pre">cmsRWSHelper</span></code> is impossible (for example because no <code class="file docutils literal notranslate"><span class="pre">cms.conf</span></code> is available) there are alternative ways to achieve the same result, presented in decreasing order of difficulty and increasing order of downtime needed.</p>
<ul class="simple">
<li>You can send a hand-crafted HTTP request to RWS (a <code class="docutils literal notranslate"><span class="pre">DELETE</span></code> method on the <code class="samp docutils literal notranslate"><span class="pre">/</span><em><span class="pre">entity_type</span></em><span class="pre">/</span><em><span class="pre">entity_id</span></em></code> resource, giving credentials by Basic Auth) and it will, all by itself, delete that object and all the ones that depend on it, recursively (that is, when deleting a task or a user it will delete its submissions and, for each of them, its subchanges).</li>
<li>You can stop RWS, delete only the JSON files of the data you want to remove and start RWS again. In this case you have to <em>manually</em> determine the depending objects and delete them as well.</li>
<li>You can stop RWS, remove <em>all</em> its data (either by deleting its data directory or by starting RWS with the <code class="docutils literal notranslate"><span class="pre">--drop</span></code> option), start RWS again and restart PS for the contest you’re interested in, to have it send the data again.</li>
</ul>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">When you change the username of an user, the name of a task or the name of a contest in CMS and then restart PS, that user, task or contest will be duplicated in RWS and you will need to delete the old copy using this procedure.</p>
</div>
</div>
<div class="section" id="multiple-contests">
<h3>Multiple contests<a class="headerlink" href="#multiple-contests" title="Permalink to this headline">¶</a></h3>
<p>Since the data in RWS will persist even after the PS that sent it has been stopped it’s possible to have many PS serve the same RWS, one after the other (or even simultaneously). This allows to have many contests inside the same RWS. The users of the contests will be merged by their username: that is, two users of two different contests will be shown as the same user if they have the same username. To show one contest at a time it’s necessary to delete the previous one before adding the next one (the procedure to delete an object is the one described in <a class="reference internal" href="#rankingwebserver-removing-data"><span class="std std-ref">Removing data</span></a>).</p>
<p>Keeping the previous contests may seem annoying to contest administrators who want to run many different and independent contests one after the other, but it’s indispensable for many-day contests like the IOI.</p>
</div>
</div>
<div class="section" id="securing-the-connection-between-ps-and-rws">
<span id="rankingwebserver-securing-the-connection-between-ss-and-rws"></span><h2>Securing the connection between PS and RWS<a class="headerlink" href="#securing-the-connection-between-ps-and-rws" title="Permalink to this headline">¶</a></h2>
<p>RWS accepts data only from clients that successfully authenticate themselves using the HTTP Basic Access Authentication. Thus an attacker that wants to alter the data on RWS needs the username and the password to authenticate its request. If they are random (and long) enough the attacker cannot guess them but may eavesdrop the plaintext HTTP request between PS and RWS. Therefore we suggest to use HTTPS, that encrypts the transmission with TLS/SSL, when the communication channel between PS and RWS is not secure.</p>
<p>HTTPS does not only protect against eavesdropping attacks but also against active attacks, like a man-in-the-middle. To do all of this it uses public-key cryptography based on so-called certificates. In our setting RWS has a public certificate (and its private key). PS has access to a copy to the same certificate and can use it to verify the identity of the receiver before sending any data (in particular before sending the username and the password!). The same certificate is then used to establish a secure communication channel.</p>
<p>The general public does not need to use HTTPS, since it is not sending nor receiving any sensitive information. We think the best solution is, for RWS, to listen on both HTTP and HTTPS ports, but to use HTTPS only for private internal use. Not having final users use HTTPS also allows you to use home-made (i.e. self-signed) certificates without causing apocalyptic warnings in the users’ browsers.</p>
<p>Note that users will still be able to connect to the HTTPS port if they discover its number, but that is of no harm. Note also that RWS will continue to accept incoming data even on the HTTP port; simply, PS will not send it.</p>
<p>To use HTTPS we suggest you to create a self-signed certificate, use that as both RWS’s and PS’s <code class="docutils literal notranslate"><span class="pre">https_certfile</span></code> and use its private key as RWS’s <code class="docutils literal notranslate"><span class="pre">https_keyfile</span></code>. If your PS manages multiple RWSs we suggest you to use a different certificate for each of those and to create a new file, obtained by joining all certificates, as the <code class="docutils literal notranslate"><span class="pre">https_certfile</span></code> of PS. Alternatively you may want to use a Certificate Authority to sign the certificates of RWSs and just give its certificate to PS. Details on how to do this follow.</p>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">Please note that, while the indications here are enough to make RWS work, computer security is a delicate subject; we urge you to be sure of what you are doing when setting up a contest in which “failure is not an option”.</p>
</div>
<div class="section" id="creating-certificates">
<h3>Creating certificates<a class="headerlink" href="#creating-certificates" title="Permalink to this headline">¶</a></h3>
<p>A quick-and-dirty way to create a self-signed certificate, ready to be used with PS and RWS, is:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>openssl req -newkey rsa:1024 -nodes -keyform PEM -keyout key.pem <span class="se">\</span>
            -new -x509 -days <span class="m">365</span> -outform PEM -out cert.pem -utf8
</pre></div>
</div>
<p>You will be prompted to enter some information to be included in the certificate. After you do this you’ll have two files, <code class="file docutils literal notranslate"><span class="pre">key.pem</span></code> and <code class="file docutils literal notranslate"><span class="pre">cert.pem</span></code>, to be used respectively as the <code class="docutils literal notranslate"><span class="pre">https_keyfile</span></code> and <code class="docutils literal notranslate"><span class="pre">https_certfile</span></code> for PS and RWS.</p>
<p>Once you have a self-signed certificate you can use it as a <abbr title="Certificate Authority">CA</abbr> to sign other certificates. If you have a <code class="docutils literal notranslate"><span class="pre">ca_key.pem</span></code>/<code class="docutils literal notranslate"><span class="pre">ca_cert.pem</span></code> pair that you want to use to create a <code class="docutils literal notranslate"><span class="pre">key.pem</span></code>/<code class="docutils literal notranslate"><span class="pre">cert.pem</span></code> pair signed by it, do:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>openssl req -newkey rsa:1024 -nodes -keyform PEM -keyout key.pem <span class="se">\</span>
            -new -outform PEM -out cert_req.pem -utf8
openssl x509 -req -in cert_req.pem -out cert.pem -days <span class="m">365</span> <span class="se">\</span>
             -CA ca_cert.pem -CAkey ca_key.pem -set_serial &lt;serial&gt;
rm cert_req.pem
</pre></div>
</div>
<p>Where <code class="docutils literal notranslate"><span class="pre">&lt;serial&gt;</span></code> is a number that has to be unique among all certificates signed by a certain CA.</p>
<p>For additional information on certificates see <a class="reference external" href="http://docs.python.org/library/ssl.html#ssl-certificates">the official Python documentation on SSL</a>.</p>
</div>
</div>
<div class="section" id="using-a-proxy">
<span id="rankingwebserver-using-a-proxy"></span><h2>Using a proxy<a class="headerlink" href="#using-a-proxy" title="Permalink to this headline">¶</a></h2>
<p>As a security measure, we recommend not to run RWS as root but to run it as an unprivileged user instead. This means that RWS cannot listen on port 80 and 443 (the default HTTP and HTTPS ports) but it needs to listen on ports whose number is higher than or equal to 1024. This is not a big issue, since we can use a reverse proxy to map the default HTTP and HTTPS ports to the ones used by RWS. We suggest you to use nginx, since it has been already proved successfully  for this purpose (some users have reported that other software, like Apache, has some issues, probably due to the use of long-polling HTTP requests by RWS).</p>
<p>A reverse proxy is most commonly used to map RWS from a high port number (say 8080) to the default HTTP port (i.e. 80), hence we will assume this scenario throughout this section.</p>
<p>With nginx it’s also extremely easy to do some URL mapping. That is, you can make RWS “share” the URL space of port 80 with other servers by making it “live” inside a prefix. This means that you will access RWS using an URL like “<a class="reference external" href="http://myserver/prefix/">http://myserver/prefix/</a>”.</p>
<p>We’ll provide here an example configuration file for nginx. This is just the “core” of the file, but other options need to be added in order for it to be complete and usable by nginx. These bits are different on each distribution, so the best is for you to take the default configuration file provided by your distribution and adapt it to contain the following code:</p>
<div class="highlight-none notranslate"><div class="highlight"><pre><span></span>http {
  server {
    listen 80;
    location ^~ /prefix/ {
      proxy_pass http://127.0.0.1:8080/;
      proxy_buffering off;
    }
  }
}
</pre></div>
</div>
<p>The trailing slash is needed in the argument of both the <code class="docutils literal notranslate"><span class="pre">location</span></code> and the <code class="docutils literal notranslate"><span class="pre">proxy_pass</span></code> option. The <code class="docutils literal notranslate"><span class="pre">proxy_buffering</span></code> option is needed for the live-update feature to work correctly (this option can be moved into <code class="docutils literal notranslate"><span class="pre">server</span></code> or <code class="docutils literal notranslate"><span class="pre">http</span></code> to give it a larger scope). To better configure how the proxy connects to RWS you can add an <code class="docutils literal notranslate"><span class="pre">upstream</span></code> section inside the <code class="docutils literal notranslate"><span class="pre">http</span></code> module, named for example <code class="docutils literal notranslate"><span class="pre">rws</span></code>, and then use <code class="docutils literal notranslate"><span class="pre">proxy_pass</span> <span class="pre">http://rws/</span></code>. This also allows you to use nginx as a load balancer in case you have many RWSs.</p>
<p>If you decide to have HTTPS for private internal use only, as suggested above (that is, you want your users to use only HTTP), then it’s perfectly fine to keep using a high port number for HTTPS and not map it to port 443, the standard HTTPS port.
Note also that you could use nginx as an HTTPS endpoint, i.e. make nginx decrypt the HTTPS trasmission and redirect it, as cleartext, into RWS’s HTTP port. This allows to use two different certificates (one by nginx, one by RWS directly), although we don’t see any real need for this.</p>
<p>The example configuration file provided in <a class="reference internal" href="index.html#running-cms-recommended-setup"><span class="std std-ref">Recommended setup</span></a> already contains sections for RWS.</p>
<div class="section" id="tuning-nginx">
<h3>Tuning nginx<a class="headerlink" href="#tuning-nginx" title="Permalink to this headline">¶</a></h3>
<p>If you’re setting up a private RWS, for internal use only, and you expect just a handful of clients then you don’t need to follow the advices given in this section. Otherwise please read on to see how to optimize nginx to handle many simultaneous connections, as required by RWS.</p>
<p>First, set the <code class="docutils literal notranslate"><span class="pre">worker_processes</span></code> option <a class="footnote-reference" href="#nginx-worker-processes" id="id1">[1]</a> of the core module to the number of CPU or cores on your machine.
Next you need to tweak the <code class="docutils literal notranslate"><span class="pre">events</span></code> module: set the <code class="docutils literal notranslate"><span class="pre">worker_connections</span></code> option <a class="footnote-reference" href="#nginx-worker-connections" id="id2">[2]</a> to a large value, at least the double of the expected number of clients divided by <code class="docutils literal notranslate"><span class="pre">worker_processes</span></code>. You could also set the <code class="docutils literal notranslate"><span class="pre">use</span></code> option <a class="footnote-reference" href="#nginx-use" id="id3">[3]</a> to an efficient event-model for your platform (like epoll on linux), but having nginx automatically decide it for you is probably better.
Then you also have to raise the maximum number of open file descriptors. Do this by setting the <code class="docutils literal notranslate"><span class="pre">worker_rlimit_nofile</span></code> option <a class="footnote-reference" href="#nginx-worker-rlimit-nofile" id="id4">[4]</a> of the core module to the same value of <code class="docutils literal notranslate"><span class="pre">worker_connections</span></code> (or greater).
You could also consider setting the <code class="docutils literal notranslate"><span class="pre">keepalive_timeout</span></code> option <a class="footnote-reference" href="#nginx-keepalive-timeout" id="id5">[5]</a> to a value like <code class="docutils literal notranslate"><span class="pre">30s</span></code>. This option can be placed inside the <code class="docutils literal notranslate"><span class="pre">http</span></code> module or inside the <code class="docutils literal notranslate"><span class="pre">server</span></code> or <code class="docutils literal notranslate"><span class="pre">location</span></code> sections, based on the scope you want to give it.</p>
<p>For more information see the official nginx documentation:</p>
<table class="docutils footnote" frame="void" id="nginx-worker-processes" rules="none">
<colgroup><col class="label" /><col /></colgroup>
<tbody valign="top">
<tr><td class="label"><a class="fn-backref" href="#id1">[1]</a></td><td><a class="reference external" href="http://wiki.nginx.org/CoreModule#worker_processes">http://wiki.nginx.org/CoreModule#worker_processes</a></td></tr>
</tbody>
</table>
<table class="docutils footnote" frame="void" id="nginx-worker-connections" rules="none">
<colgroup><col class="label" /><col /></colgroup>
<tbody valign="top">
<tr><td class="label"><a class="fn-backref" href="#id2">[2]</a></td><td><a class="reference external" href="http://wiki.nginx.org/EventsModule#worker_connections">http://wiki.nginx.org/EventsModule#worker_connections</a></td></tr>
</tbody>
</table>
<table class="docutils footnote" frame="void" id="nginx-use" rules="none">
<colgroup><col class="label" /><col /></colgroup>
<tbody valign="top">
<tr><td class="label"><a class="fn-backref" href="#id3">[3]</a></td><td><a class="reference external" href="http://wiki.nginx.org/EventsModule#use">http://wiki.nginx.org/EventsModule#use</a></td></tr>
</tbody>
</table>
<table class="docutils footnote" frame="void" id="nginx-worker-rlimit-nofile" rules="none">
<colgroup><col class="label" /><col /></colgroup>
<tbody valign="top">
<tr><td class="label"><a class="fn-backref" href="#id4">[4]</a></td><td><a class="reference external" href="http://wiki.nginx.org/CoreModule#worker_rlimit_nofile">http://wiki.nginx.org/CoreModule#worker_rlimit_nofile</a></td></tr>
</tbody>
</table>
<table class="docutils footnote" frame="void" id="nginx-keepalive-timeout" rules="none">
<colgroup><col class="label" /><col /></colgroup>
<tbody valign="top">
<tr><td class="label"><a class="fn-backref" href="#id5">[5]</a></td><td><a class="reference external" href="http://wiki.nginx.org/HttpCoreModule#keepalive_timeout">http://wiki.nginx.org/HttpCoreModule#keepalive_timeout</a></td></tr>
</tbody>
</table>
</div>
</div>
<div class="section" id="some-final-suggestions">
<h2>Some final suggestions<a class="headerlink" href="#some-final-suggestions" title="Permalink to this headline">¶</a></h2>
<p>The suggested setup (the one that we also used at the IOI 2012) is to make RWS listen on both HTTP and HTTPS ports (we used 8080 and 8443), to use nginx to map port 80 to port 8080, to make all three ports (80, 8080 and 8443) accessible from the internet, to make PS connect to RWS via HTTPS on port 8443 and to use a Certificate Authority to generate certificates (the last one is probably an overkill).</p>
<p>At the IOI 2012, we had only one server, running on a 2 GHz machine, and we were able to serve about 1500 clients simultaneously (and, probably, we were limited to this value by a misconfiguration of nginx). This is to say that you’ll likely need only one public RWS server.</p>
<p>If you’re starting RWS on your server remotely, for example via SSH, make sure the <code class="docutils literal notranslate"><span class="pre">screen</span></code> command is your friend :-).</p>
</div>
</div>
<span id="document-Localization"></span><div class="section" id="localization">
<span id="id1"></span><h1>Localization<a class="headerlink" href="#localization" title="Permalink to this headline">¶</a></h1>
<div class="section" id="for-developers">
<h2>For developers<a class="headerlink" href="#for-developers" title="Permalink to this headline">¶</a></h2>
<p>When you change a string in a template or in a web server, you have to generate again the file <a class="reference external" href="https://github.com/cms-dev/cms/blob/v1.5.dev0/cms/locale/cms.pot">cms/locale/cms.pot</a>. To do so, run this command from the root of the repository.</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>./setup.py extract_messages
</pre></div>
</div>
<p>When you have a new translation, or an update of an old translation, you need to update the <code class="docutils literal notranslate"><span class="pre">cms.mo</span></code> files (the compiled versions of the <code class="docutils literal notranslate"><span class="pre">cms.po</span></code> files). You can run <code class="docutils literal notranslate"><span class="pre">python</span> <span class="pre">setup.py</span> <span class="pre">compile_catalog</span></code> to update the <code class="docutils literal notranslate"><span class="pre">cms.mo</span></code> files for all translations (or add the <code class="docutils literal notranslate"><span class="pre">-l</span> <span class="pre">&lt;code&gt;</span></code> argument to update only the one for a given locale). The usual <code class="docutils literal notranslate"><span class="pre">python</span> <span class="pre">setup.py</span> <span class="pre">install</span></code> will do this automatically. Note that, to have the new strings, you need to restart the web server.</p>
</div>
<div class="section" id="for-translators">
<h2>For translators<a class="headerlink" href="#for-translators" title="Permalink to this headline">¶</a></h2>
<p>To begin translating to a new language, run this command from the root of the repository.</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>./setup.py init_catalog -l &lt;code&gt;
</pre></div>
</div>
<p>Right after that, open the newly created <code class="file docutils literal notranslate"><span class="pre">cms/locale/&lt;code&gt;/LC_MESSAGES/cms.po</span></code> and fill the information in the header. To translate a string, simply fill the corresponding msgstr with the translations. You can also use specialized translation softwares such as poEdit and others.</p>
<p>When the developers update the <code class="docutils literal notranslate"><span class="pre">cms.pot</span></code> file, you do not need to start from scratch. Instead, you can create a new <code class="docutils literal notranslate"><span class="pre">cms.po</span></code> file that merges the old translated string with the new, to-be-translated ones. The command is the following, run inside the root of the repository.</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>./setup.py update_catalog -l &lt;code&gt;
</pre></div>
</div>
<p>After you are done translating the messages, please run the following command and check that no error messages are reported (the developers will be glad to assist you if any of them isn’t clear):</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>./setup.py compile_catalog -l &lt;code&gt;
</pre></div>
</div>
</div>
</div>
<span id="document-Troubleshooting"></span><div class="section" id="troubleshooting">
<h1>Troubleshooting<a class="headerlink" href="#troubleshooting" title="Permalink to this headline">¶</a></h1>
<p>Subtle issues with CMS can arise from old versions of libraries or supporting software. Please ensure you are running the minimum versions of each dependency (described in <a class="reference internal" href="index.html#installation-dependencies"><span class="std std-ref">Dependencies and available compilers</span></a>).</p>
<p>In the next sections we list some known symptoms and their possible causes.</p>
<div class="section" id="database">
<h2>Database<a class="headerlink" href="#database" title="Permalink to this headline">¶</a></h2>
<ul>
<li><p class="first"><em>Symptom.</em> Error message “Cannot determine OID of function lo_create”</p>
<p><em>Possible cause.</em> Your database must be at least PostgreSQL 8.x to support large objects used by CMS.</p>
</li>
<li><p class="first"><em>Symptom.</em> Exceptions regarding missing database fields or with the wrong type.</p>
<p><em>Possible cause.</em> The version of CMS that created the schema in your database is different from the one you are using now. If the schema is older than the current version, you can update the schema as in <a class="reference internal" href="index.html#installation-updatingcms"><span class="std std-ref">Installing CMS and its Python dependencies</span></a>.</p>
</li>
<li><p class="first"><em>Symptom.</em> Some components of CMS fail randomly and PostgreSQL complains about having too many connections.</p>
<p><em>Possible cause.</em> The default configuration of PostgreSQL may allow insufficiently many incoming connections on the database engine. You can raise this limit by tweaking the <code class="docutils literal notranslate"><span class="pre">`max_connections`</span></code> parameter in <code class="docutils literal notranslate"><span class="pre">`postgresql.conf`</span></code> (<a class="reference external" href="http://www.postgresql.org/docs/9.1/static/runtime-config-connection.html">see docs</a>). This, in turn, requires more shared memory for the PostgreSQL process (see <code class="docutils literal notranslate"><span class="pre">`shared_buffers`</span></code> parameter in <a class="reference external" href="http://www.postgresql.org/docs/9.1/static/runtime-config-resource.html">docs</a>), which may overflow the maximum limit allowed by the operating system. In such case see the suggestions in <a class="reference external" href="http://www.postgresql.org/docs/9.1/static/kernel-resources.html#SYSVIPC">http://www.postgresql.org/docs/9.1/static/kernel-resources.html#SYSVIPC</a>. Users reported that another way to go is to use a connection pooler like <a class="reference external" href="https://wiki.postgresql.org/wiki/PgBouncer">PgBouncer</a>.</p>
</li>
</ul>
</div>
<div class="section" id="services">
<h2>Services<a class="headerlink" href="#services" title="Permalink to this headline">¶</a></h2>
<ul>
<li><p class="first"><em>Symptom.</em> Some services log error messages like <code class="samp docutils literal notranslate"><span class="pre">Response</span> <span class="pre">is</span> <span class="pre">missing</span> <span class="pre">some</span> <span class="pre">fields,</span> <span class="pre">ignoring</span></code> then disconnect from other services.</p>
<p><em>Possible cause.</em> It is possible that a service that was trying to establish a connection to another service residing on the same host was assigned by the kernel an outgoing port that is equal to the port it was trying to reach. This can be verified by looking for logs that resemble the following: <code class="samp docutils literal notranslate"><span class="pre">Established</span> <span class="pre">connection</span> <span class="pre">with</span> <span class="pre">192.168.1.1:43210</span> <span class="pre">(LogService,0)</span> <span class="pre">(local</span> <span class="pre">address:</span> <span class="pre">192.168.1.1:43210)</span></code> (observe the same address repeated twice).</p>
<p>A workaround for this issue is to first look at what range of ports is reserved by the kernel to “ephemeral” ports (the ones dynamically assigned to outgoing connections). This can be found out with <code class="docutils literal notranslate"><span class="pre">cat</span> <span class="pre">/proc/sys/net/ipv4/ip_local_port_range</span></code>. Then the configuration file of CMS should be updated so that all services are assigned ports outside that range.</p>
</li>
</ul>
</div>
<div class="section" id="servers">
<h2>Servers<a class="headerlink" href="#servers" title="Permalink to this headline">¶</a></h2>
<ul>
<li><p class="first"><em>Symptom.</em> Some HTTP requests to ContestWebServer take a long time and fail with 500 Internal Server Error. ContestWebServer logs contain entries such as <code class="samp docutils literal notranslate"><span class="pre">TimeoutError('QueuePool</span> <span class="pre">limit</span> <span class="pre">of</span> <span class="pre">size</span> <span class="pre">5</span> <span class="pre">overflow</span> <span class="pre">10</span> <span class="pre">reached,</span> <span class="pre">connection</span> <span class="pre">timed</span> <span class="pre">out,</span> <span class="pre">timeout</span> <span class="pre">60',)</span></code>.</p>
<p><em>Possible cause.</em> The server may be overloaded with user requests. You can try to increase the <code class="docutils literal notranslate"><span class="pre">`pool_timeout`</span></code> argument in <a class="reference external" href="https://github.com/cms-dev/cms/blob/v1.5.dev0/cms/db/__init__.py">cms/db/__init__.py</a> or, preferably, spread your users over more instances of ContestWebServer.</p>
</li>
<li><p class="first"><em>Symptom.</em> Message from ContestWebServer such as: <code class="samp docutils literal notranslate"><span class="pre">WARNING:root:Invalid</span> <span class="pre">cookie</span> <span class="pre">signature</span> <span class="pre">KFZzdW5kdWRlCnAwCkkxMzI5MzQzNzIwCnRw...</span></code></p>
<p><em>Possible cause.</em> The contest secret key (defined in cms.conf) may have been changed and users’ browsers are still attempting to use cookies signed with the old key. If this is the case, the problem should correct itself and won’t be seen by users.</p>
</li>
<li><p class="first"><em>Symptom.</em> Ranking Web Server displays wrong data, or too much data.</p>
<p><em>Possible cause.</em> RWS is designed to handle groups of contests, so it retains data about past contests. If you want to delete previous data, run RWS with the <code class="docutils literal notranslate"><span class="pre">`-d`</span></code> option. See <a class="reference internal" href="index.html#document-RankingWebServer"><span class="doc">RankingWebServer</span></a> for more details</p>
</li>
<li><p class="first"><em>Symptom.</em> Ranking Web Server prints an “Inconsistent data” exception.</p>
<p><em>Possible cause.</em> RWS has its own local storage of the score data; this exception usually means that it got corrupted in some way (e.g., some of the data was deleted). If all the scores are still present in the core CMS, the easiest way to fix this is to stop RWS and ProxyService, run <code class="docutils literal notranslate"><span class="pre">cmsRankingWebServer</span> <span class="pre">-d</span></code> to delete the local storage, then start again RWS and PS.</p>
</li>
</ul>
</div>
<div class="section" id="sandbox">
<h2>Sandbox<a class="headerlink" href="#sandbox" title="Permalink to this headline">¶</a></h2>
<ul>
<li><p class="first"><em>Symptom.</em> The Worker fails to evaluate a submission logging about an invalid (empty) output from the manager.</p>
<p><em>Possible cause.</em> You might have been used a non-statically linked checker. The sandbox prevent dynamically linked executables to work. Try compiling the checker with <code class="docutils literal notranslate"><span class="pre">`-static`</span></code>. Also, make sure that the checker was compiled for the architecture of the workers (e.g., 32 or 64 bits).</p>
</li>
<li><p class="first"><em>Symptom.</em> The Worker fails to evaluate a submission with a generic failure.</p>
<p><em>Possible cause.</em> Make sure that the isolate binary that CMS is using has the correct permissions (in particular, its owner is root and it has the suid bit set). Be careful of having multiple isolate binaries in your path. Another reason could be that you are using an old version of isolate.</p>
</li>
<li><p class="first"><em>Symptom.</em> Contestants’ solutions fail when trying to write large outputs.</p>
<p><em>Possible cause.</em> CMS limits the maximum output size from programs being evaluated for security reasons. Currently the limit is 1 GB and can be configured by changing the parameter <code class="docutils literal notranslate"><span class="pre">max_file_size</span></code> in <code class="file docutils literal notranslate"><span class="pre">cms.conf</span></code>.</p>
</li>
</ul>
</div>
<div class="section" id="evaluations">
<h2>Evaluations<a class="headerlink" href="#evaluations" title="Permalink to this headline">¶</a></h2>
<ul>
<li><p class="first"><em>Symptom.</em> Submissions that should  exceed memory limit actually pass or exceed the time limits.</p>
<p><em>Possible cause.</em> You have an active swap partition on the workers; isolate only limit physical memory, not swap usage. Disable the swap with <code class="docutils literal notranslate"><span class="pre">sudo</span> <span class="pre">swapoff</span> <span class="pre">-a</span></code>.</p>
</li>
<li><p class="first"><em>Symptom.</em> Re-running evaluations gives very different time or memory usage.</p>
<p><em>Possible cause.</em> Make sure the workers are configured in a way to minimize resource usage variability, by following isolate’s <a class="reference external" href="https://github.com/ioi/isolate/blob/c679ae936d8e8d64e5dab553bdf1b22261324315/isolate.1.txt#L292">guidelines</a> for reproducible results.</p>
</li>
</ul>
</div>
</div>
<span id="document-Internals"></span><div class="section" id="internals">
<h1>Internals<a class="headerlink" href="#internals" title="Permalink to this headline">¶</a></h1>
<p>This section contains some details about some CMS internals. They are
mostly meant for developers, not for users. However, if you are curious
about what’s under the hood, you will find something interesting here
(though without any pretension of completeness). Moreover, these are
not meant to be full specifications, but only useful notes for the
future.</p>
<p>Oh, I was nearly forgetting: if you are curious about what happens
inside CMS, you may actually be interested in helping us writing
it. We can assure you it is a very rewarding task. After all, if you
are hanging around here, you must have some interest in coding! In
case, feel free <a class="reference external" href="http://cms-dev.github.io/">to get in touch with us</a>.</p>
<div class="section" id="rpc-protocol">
<h2>RPC protocol<a class="headerlink" href="#rpc-protocol" title="Permalink to this headline">¶</a></h2>
<p>Different CMS processes communicate between them by mean of TCP
sockets. Once a service has established a socket with another, it can
write messages on the stream; each message is a JSON-encoded object,
terminated by a <code class="docutils literal notranslate"><span class="pre">\r\n</span></code> string (this, of course, means that <code class="docutils literal notranslate"><span class="pre">\r\n</span></code>
cannot be used in the JSON encoding: this is not a problem, since new
lines inside string represented in the JSON have to be escaped
anyway).</p>
<p>An RPC request must be of the form (it is pretty printed here, but it
is sent in compact form inside CMS):</p>
<div class="highlight-default notranslate"><div class="highlight"><pre><span></span><span class="p">{</span>
  <span class="s2">&quot;__method&quot;</span><span class="p">:</span> <span class="o">&lt;</span><span class="n">name</span> <span class="n">of</span> <span class="n">the</span> <span class="n">requested</span> <span class="n">method</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="s2">&quot;__data&quot;</span><span class="p">:</span> <span class="p">{</span>
              <span class="o">&lt;</span><span class="n">name</span> <span class="n">of</span> <span class="n">first</span> <span class="n">arg</span><span class="o">&gt;</span><span class="p">:</span> <span class="o">&lt;</span><span class="n">value</span> <span class="n">of</span> <span class="n">first</span> <span class="n">arg</span><span class="o">&gt;</span><span class="p">,</span>
              <span class="o">...</span>
            <span class="p">},</span>
  <span class="s2">&quot;__id&quot;</span><span class="p">:</span> <span class="o">&lt;</span><span class="n">random</span> <span class="n">ID</span> <span class="n">string</span><span class="o">&gt;</span>
<span class="p">}</span>
</pre></div>
</div>
<p>The arguments in <code class="docutils literal notranslate"><span class="pre">__data</span></code> are (of course) not ordered: they have to
be matched according to their names. In particular, this means that
our protocol enables us to use a <code class="docutils literal notranslate"><span class="pre">kwargs</span></code>-like interface, but not a
<code class="docutils literal notranslate"><span class="pre">args</span></code>-like one. That’s not so terrible, anyway.</p>
<p>The <code class="docutils literal notranslate"><span class="pre">__id</span></code> is a random string that will be returned in the response,
and it is useful (actually, it’s the only way) to match requests with
responses.</p>
<p>The response is of the form:</p>
<div class="highlight-default notranslate"><div class="highlight"><pre><span></span><span class="p">{</span>
  <span class="s2">&quot;__data&quot;</span><span class="p">:</span> <span class="o">&lt;</span><span class="k">return</span> <span class="n">value</span> <span class="ow">or</span> <span class="n">null</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="s2">&quot;__error&quot;</span><span class="p">:</span> <span class="o">&lt;</span><span class="n">null</span> <span class="ow">or</span> <span class="n">error</span> <span class="n">string</span><span class="o">&gt;</span><span class="p">,</span>
  <span class="s2">&quot;__id&quot;</span><span class="p">:</span> <span class="o">&lt;</span><span class="n">random</span> <span class="n">ID</span> <span class="n">string</span><span class="o">&gt;</span>
<span class="p">}</span>
</pre></div>
</div>
<p>The value of <code class="docutils literal notranslate"><span class="pre">__id</span></code> must of course be the same as in the request.
If <code class="docutils literal notranslate"><span class="pre">__error</span></code> is not null, then <code class="docutils literal notranslate"><span class="pre">__data</span></code> is expected to be null.</p>
</div>
<div class="section" id="backdoor">
<h2>Backdoor<a class="headerlink" href="#backdoor" title="Permalink to this headline">¶</a></h2>
<p>Setting the <code class="docutils literal notranslate"><span class="pre">backdoor</span></code> configuration key to true causes services to
serve a Python console (accessible with netcat), running in the same
interpreter instance as the service, allowing to inspect and modify its
data, live. It will be bound to a local UNIX domain socket, usually at
<code class="file docutils literal notranslate"><span class="pre">/var/local/run/cms/</span><em><span class="pre">service</span></em><span class="pre">_</span><em><span class="pre">shard</span></em></code>. Access is granted only to
users belonging to the cmsuser group.
Although there’s no authentication mechanism to prevent unauthorized
access, the restrictions on the file should make it safe to run the
backdoor everywhere, even on workers that are used as contestants’
machines.
You can use <code class="docutils literal notranslate"><span class="pre">rlwrap</span></code> to add basic readline support. For example, the
following is a complete working connection command:</p>
<div class="highlight-bash notranslate"><div class="highlight"><pre><span></span>rlwrap netcat -U /var/local/run/cms/EvaluationService_0
</pre></div>
</div>
<p>Substitute <code class="docutils literal notranslate"><span class="pre">netcat</span></code> with your implementation (<code class="docutils literal notranslate"><span class="pre">nc</span></code>, <code class="docutils literal notranslate"><span class="pre">ncat</span></code>, etc.)
if needed.</p>
</div>
</div>
</div>
</div>


          </div>
          
        </div>
      </div>
      <div class="sphinxsidebar" role="navigation" aria-label="main navigation">
        <div class="sphinxsidebarwrapper">
<h1 class="logo"><a href="index.html#document-index">CMS</a></h1>








<h2>Navigation</h2>
<ul>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Introduction">Introduction</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Installation">Installation</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Running CMS">Running CMS</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Data model">Data model</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Creating a contest">Creating a contest</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Configuring a contest">Configuring a contest</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Detailed timing configuration">Detailed timing configuration</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Task types">Task types</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Score types">Score types</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Task versioning">Task versioning</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-External contest formats">External contest formats</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-RankingWebServer">RankingWebServer</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Localization">Localization</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Troubleshooting">Troubleshooting</a></li>
<li class="toctree-l1"><a class="reference internal" href="index.html#document-Internals">Internals</a></li>
</ul>

<div class="relations">
<h2>Related Topics</h2>
<ul>
  <li><a href="index.html#document-index">Documentation overview</a><ul>
  </ul></li>
</ul>
</div>
<div id="searchbox" style="display: none" role="search">
  <h2>Quick search</h2>
    <div class="searchformwrapper">
    <form class="search" action="search.html" method="get">
      <input type="text" name="q" />
      <input type="submit" value="Go" />
      <input type="hidden" name="check_keywords" value="yes" />
      <input type="hidden" name="area" value="default" />
    </form>
    </div>
</div>
<script type="text/javascript">$('#searchbox').show(0);</script>








        </div>
      </div>
      <div class="clearer"></div>
    </div>
    <div class="footer">
      &copy;2020, The CMS development team.
      
      |
      Powered by <a href="http://sphinx-doc.org/">Sphinx 1.8.5</a>
      &amp; <a href="https://github.com/bitprophet/alabaster">Alabaster 0.7.12</a>
      
    </div>

    

    
  </body>
</html>
</dl>