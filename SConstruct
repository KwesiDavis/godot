EnsureSConsVersion(0,14);

import string
import os
import os.path
import glob
import sys
import methods

methods.update_version()

# scan possible build platforms

platform_list = [] # list of platforms
platform_opts = {} # options for each platform
platform_flags = {} # flags for each platform


active_platforms=[]
active_platform_ids=[]
platform_exporters=[]
global_defaults=[]

for x in glob.glob("platform/*"):
	if (not os.path.isdir(x)):
		continue
	tmppath="./"+x

	sys.path.append(tmppath)
	import detect

	if (os.path.exists(x+"/export/export.cpp")):
		platform_exporters.append(x[9:])
	if (os.path.exists(x+"/globals/global_defaults.cpp")):
		global_defaults.append(x[9:])
	if (detect.is_active()):
		active_platforms.append( detect.get_name() )
		active_platform_ids.append(x);
	if (detect.can_build()):
		x=x.replace("platform/","") # rest of world
		x=x.replace("platform\\","") # win32
		platform_list+=[x]
		platform_opts[x]=detect.get_opts()
		platform_flags[x]=detect.get_flags()
	sys.path.remove(tmppath)
	sys.modules.pop('detect')

module_list=methods.detect_modules()


#print "Detected Platforms: "+str(platform_list)

methods.save_active_platforms(active_platforms,active_platform_ids)

custom_tools=['default']

platform_arg = ARGUMENTS.get("platform", False)

if (os.name=="posix"):
	pass
elif (os.name=="nt"):
	if (os.getenv("VSINSTALLDIR")==None or platform_arg=="android"):
		custom_tools=['mingw']

env_base=Environment(tools=custom_tools,ENV = {'PATH' : os.environ['PATH']});

#env_base=Environment(tools=custom_tools);
env_base.global_defaults=global_defaults
env_base.android_source_modules=[]
env_base.android_source_files=[]
env_base.android_module_libraries=[]
env_base.android_manifest_chunk=""
env_base.android_permission_chunk=""
env_base.android_appattributes_chunk=""
env_base.disabled_modules=[]

env_base.__class__.android_module_source = methods.android_module_source
env_base.__class__.android_module_library = methods.android_module_library
env_base.__class__.android_module_file = methods.android_module_file
env_base.__class__.android_module_manifest = methods.android_module_manifest
env_base.__class__.android_module_permission = methods.android_module_permission
env_base.__class__.android_module_attribute = methods.android_module_attribute
env_base.__class__.disable_module = methods.disable_module

env_base.__class__.add_source_files = methods.add_source_files

env_base["x86_opt_gcc"]=False
env_base["x86_opt_vc"]=False
env_base["armv7_opt_gcc"]=False

customs = ['custom.py']

profile = ARGUMENTS.get("profile", False)
if profile:
	import os.path
	if os.path.isfile(profile):
		customs.append(profile)
	elif os.path.isfile(profile+".py"):
		customs.append(profile+".py")

opts=Variables(customs, ARGUMENTS)
opts.Add('target', 'Compile Target (debug/release_debug/release).', "debug")
opts.Add('bits', 'Compile Target Bits (default/32/64).', "default")
opts.Add('platform','Platform: '+str(platform_list)+'.',"")
opts.Add('p','Platform (same as platform=).',"")
opts.Add('tools','Build Tools (Including Editor): (yes/no)','yes')
opts.Add('gdscript','Build GDSCript support: (yes/no)','yes')
opts.Add('vorbis','Build Ogg Vorbis Support: (yes/no)','yes')
opts.Add('opus','Build Opus Audio Format Support: (yes/no)','yes')
opts.Add('minizip','Build Minizip Archive Support: (yes/no)','yes')
opts.Add('squish','Squish BC Texture Compression in editor (yes/no)','yes')
opts.Add('theora','Theora Video (yes/no)','yes')
opts.Add('theoralib','Theora Video (yes/no)','no')
opts.Add('freetype','Freetype support in editor','yes')
opts.Add('speex','Speex Audio (yes/no)','yes')
opts.Add('xml','XML Save/Load support (yes/no)','yes')
opts.Add('png','PNG Image loader support (yes/no)','yes')
opts.Add('jpg','JPG Image loader support (yes/no)','yes')
opts.Add('webp','WEBP Image loader support (yes/no)','yes')
opts.Add('dds','DDS Texture loader support (yes/no)','yes')
opts.Add('pvr','PVR (PowerVR) Texture loader support (yes/no)','yes')
opts.Add('etc1','etc1 Texture compression support (yes/no)','yes')
opts.Add('builtin_zlib','Use built-in zlib (yes/no)','yes')
opts.Add('openssl','Use OpenSSL (yes/no/builtin)','no')
opts.Add('musepack','Musepack Audio (yes/no)','yes')
opts.Add("CXX", "Compiler");
opts.Add("CCFLAGS", "Custom flags for the C++ compiler");
opts.Add("CFLAGS", "Custom flags for the C compiler");
opts.Add("LINKFLAGS", "Custom flags for the linker");
opts.Add('disable_3d', 'Disable 3D nodes for smaller executable (yes/no)', "no")
opts.Add('disable_advanced_gui', 'Disable advance 3D gui nodes and behaviors (yes/no)', "no")
opts.Add('colored', 'Enable colored output for the compilation (yes/no)', 'no')
opts.Add('extra_suffix', 'Custom extra suffix added to the base filename of all generated binary files.', '')
opts.Add('vsproj', 'Generate Visual Studio Project. (yes/no)', 'no')

# add platform specific options

for k in platform_opts.keys():
	opt_list = platform_opts[k]
	for o in opt_list:
		opts.Add(o[0],o[1],o[2])

for x in module_list:
	opts.Add('module_'+x+'_enabled', "Enable module '"+x+"'.", "yes")

opts.Update(env_base) # update environment
Help(opts.GenerateHelpText(env_base)) # generate help

# add default include paths

env_base.Append(CPPPATH=['#core','#core/math','#tools','#drivers','#'])

# configure ENV for platform
env_base.platform_exporters=platform_exporters

"""
sys.path.append("./platform/"+env_base["platform"])
import detect
detect.configure(env_base)
sys.path.remove("./platform/"+env_base["platform"])
sys.modules.pop('detect')
"""

if (env_base['target']=='debug'):
	env_base.Append(CPPFLAGS=['-DDEBUG_MEMORY_ALLOC']);
	env_base.Append(CPPFLAGS=['-DSCI_NAMESPACE'])

env_base.platforms = {}


selected_platform =""

if env_base['platform'] != "":
	selected_platform=env_base['platform']
elif env_base['p'] != "":
	selected_platform=env_base['p']
	env_base["platform"]=selected_platform


if selected_platform in platform_list:

	sys.path.append("./platform/"+selected_platform)
	import detect
	if "create" in dir(detect):
		env = detect.create(env_base)
	else:
		env = env_base.Clone()

	if env['vsproj']=="yes":
		env.vs_incs = []
		env.vs_srcs = []

		def AddToVSProject( sources ):
			for x in sources:
				if type(x) == type(""):
					fname = env.File(x).path
				else:
					fname = env.File(x)[0].path
				pieces =  fname.split(".")
				if len(pieces)>0:
					basename = pieces[0]
					basename = basename.replace('\\\\','/')
					env.vs_srcs = env.vs_srcs + [basename + ".cpp"]
					env.vs_incs = env.vs_incs + [basename + ".h"]
					#print basename
		env.AddToVSProject = AddToVSProject

	env.extra_suffix=""

	if env["extra_suffix"] != '' :
		env.extra_suffix += '.'+env["extra_suffix"]

	CCFLAGS = env.get('CCFLAGS', '')
	env['CCFLAGS'] = ''

	env.Append(CCFLAGS=string.split(str(CCFLAGS)))

	CFLAGS = env.get('CFLAGS', '')
	env['CFLAGS'] = ''

	env.Append(CFLAGS=string.split(str(CFLAGS)))

	LINKFLAGS = env.get('LINKFLAGS', '')
	env['LINKFLAGS'] = ''

	env.Append(LINKFLAGS=string.split(str(LINKFLAGS)))

	flag_list = platform_flags[selected_platform]
	for f in flag_list:
		if not (f[0] in ARGUMENTS): # allow command line to override platform flags
			env[f[0]] = f[1]

	#must happen after the flags, so when flags are used by configure, stuff happens (ie, ssl on x11)
	detect.configure(env)

	#env['platform_libsuffix'] = env['LIBSUFFIX']

	suffix="."+selected_platform

	if (env["target"]=="release"):
		if (env["tools"]=="yes"):
			print("Tools can only be built with targets 'debug' and 'release_debug'.")
			sys.exit(255)
		suffix+=".opt"

	elif (env["target"]=="release_debug"):
		if (env["tools"]=="yes"):
			suffix+=".opt.tools"
		else:
			suffix+=".opt.debug"
	else:
		if (env["tools"]=="yes"):
			suffix+=".tools"
		else:
			suffix+=".debug"

	if (env["bits"]=="32"):
		suffix+=".32"
	elif (env["bits"]=="64"):
		suffix+=".64"

	suffix+=env.extra_suffix

	env["PROGSUFFIX"]=suffix+env["PROGSUFFIX"]
	env["OBJSUFFIX"]=suffix+env["OBJSUFFIX"]
	env["LIBSUFFIX"]=suffix+env["LIBSUFFIX"]
	env["SHLIBSUFFIX"]=suffix+env["SHLIBSUFFIX"]

	sys.path.remove("./platform/"+selected_platform)
	sys.modules.pop('detect')


	env.module_list=[]

	for x in module_list:
		if env['module_'+x+'_enabled'] != "yes":
			continue
		tmppath="./modules/"+x
		sys.path.append(tmppath)
		env.current_module=x
		import config
		if (config.can_build(selected_platform)):
			config.configure(env)
			env.module_list.append(x)
		sys.path.remove(tmppath)
		sys.modules.pop('config')


	if (env['musepack']=='yes'):
		env.Append(CPPFLAGS=['-DMUSEPACK_ENABLED']);

	if (env['openssl']!='no'):
		env.Append(CPPFLAGS=['-DOPENSSL_ENABLED']);
		if (env['openssl']=="builtin"):
			env.Append(CPPPATH=['#drivers/builtin_openssl2'])

	if (env["builtin_zlib"]=='yes'):
		env.Append(CPPPATH=['#drivers/builtin_zlib/zlib'])

	# to test 64 bits compiltion
	# env.Append(CPPFLAGS=['-m64'])

	if (env_base['squish']=='yes'):
		env.Append(CPPFLAGS=['-DSQUISH_ENABLED']);

	if (env['vorbis']=='yes'):
		env.Append(CPPFLAGS=['-DVORBIS_ENABLED']);
	if (env['opus']=='yes'):
		env.Append(CPPFLAGS=['-DOPUS_ENABLED']);


	if (env['theora']=='yes'):
		env['theoralib']='yes'
		env.Append(CPPFLAGS=['-DTHEORA_ENABLED']);		
	if (env['theoralib']=='yes'):
		env.Append(CPPFLAGS=['-DTHEORALIB_ENABLED']);

	if (env['png']=='yes'):
		env.Append(CPPFLAGS=['-DPNG_ENABLED']);
	if (env['dds']=='yes'):
		env.Append(CPPFLAGS=['-DDDS_ENABLED']);
	if (env['pvr']=='yes'):
		env.Append(CPPFLAGS=['-DPVR_ENABLED']);
	if (env['jpg']=='yes'):
		env.Append(CPPFLAGS=['-DJPG_ENABLED']);
	if (env['webp']=='yes'):
		env.Append(CPPFLAGS=['-DWEBP_ENABLED']);

	if (env['speex']=='yes'):
		env.Append(CPPFLAGS=['-DSPEEX_ENABLED']);

	if (env['tools']=='yes'):
		env.Append(CPPFLAGS=['-DTOOLS_ENABLED'])
	if (env['disable_3d']=='yes'):
		env.Append(CPPFLAGS=['-D_3D_DISABLED'])
	if (env['gdscript']=='yes'):
		env.Append(CPPFLAGS=['-DGDSCRIPT_ENABLED'])
	if (env['disable_advanced_gui']=='yes'):
		env.Append(CPPFLAGS=['-DADVANCED_GUI_DISABLED'])

	if (env['minizip'] == 'yes'):
		env.Append(CPPFLAGS=['-DMINIZIP_ENABLED'])

	if (env['xml']=='yes'):
		env.Append(CPPFLAGS=['-DXML_ENABLED'])

	if (env['colored']=='yes'):
		methods.colored(sys,env)

	if (env['etc1']=='yes'):
		env.Append(CPPFLAGS=['-DETC1_ENABLED'])

	Export('env')

	#build subdirs, the build order is dependent on link order.

	SConscript("core/SCsub")
	SConscript("servers/SCsub")
	SConscript("scene/SCsub")
	SConscript("tools/SCsub")
	SConscript("drivers/SCsub")
	SConscript("bin/SCsub")

	SConscript("modules/SCsub")
	SConscript("main/SCsub")

	SConscript("platform/"+selected_platform+"/SCsub"); # build selected platform

	# Microsoft Visual Studio Project Generation
	if (env['vsproj'])=="yes":

		AddToVSProject(env.core_sources)
		AddToVSProject(env.main_sources)
		AddToVSProject(env.modules_sources)
		AddToVSProject(env.scene_sources)
		AddToVSProject(env.servers_sources)
		AddToVSProject(env.tool_sources)

		#env['MSVS_VERSION']='9.0'
		env['MSVSBUILDCOM'] = "scons platform=" + selected_platform + " target=" + env["target"] + " bits=" + env["bits"] + " tools=yes"
		env['MSVSREBUILDCOM'] = "scons platform=" + selected_platform + " target=" + env["target"] + " bits=" + env["bits"] + " tools=yes vsproj=true"
		env['MSVSCLEANCOM'] = "scons --clean platform=" + selected_platform + " target=" + env["target"] + " bits=" + env["bits"] + " tools=yes"

		debug_variants = ['Debug|Win32']+['Debug|x64']
		release_variants = ['Release|Win32']+['Release|x64']
		release_debug_variants = ['Release_Debug|Win32']+['Release_Debug|x64']
		variants = debug_variants + release_variants + release_debug_variants
		debug_targets = ['Debug']+['Debug']
		release_targets = ['Release']+['Release']
		release_debug_targets = ['ReleaseDebug']+['ReleaseDebug']
		targets = debug_targets + release_targets + release_debug_targets
		msvproj = env.MSVSProject(target = ['#godot' + env['MSVSPROJECTSUFFIX'] ],
								incs = env.vs_incs,
								srcs = env.vs_srcs,
								runfile = targets,
								buildtarget = targets,
								auto_build_solution=1,
								variant = variants)

else:

	print("No valid target platform selected.")
	print("The following were detected:")
	for x in platform_list:
		print("\t"+x)
	print("\nPlease run scons again with argument: platform=<string>")
