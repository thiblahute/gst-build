project('All GStreamer modules', 'c', version : '1.13.0.1', meson_version : '>= 0.40.1')

gst_version = '>= @0@'.format(meson.project_version())
gst_branch = 'master'

glib_req = '>= 2.40.0'

subprojects = [
    'gstreamer',
    'gst-plugins-base',
    'gst-plugins-good',
]

cc = meson.get_compiler('c')
if not meson.is_subproject() and cc.get_id() == 'msvc'
  # Make it possible to use msys2 built zlib
  # which fails when not using the mingw toolchain as
  # it uses unistd.h
  add_global_arguments('-DZ_SOLO', language: 'c')
endif

# FIXME Remove that check once we have ffmpeg as a gst-libav subproject
libavfilter_dep = dependency('libavfilter', version: '>= 6.47.100', required: false)
gst_libav = []
if libavfilter_dep.found()
    cc = meson.get_compiler('c')
    check_ffmpeg_src = '''#include <libavcodec/avcodec.h>
    #if LIBAVCODEC_VERSION_MICRO >= 100
    /* FFmpeg uses 100+ as its micro version */
    #else
    #error libav provider should be FFmpeg
    #endif'''
    if cc.compiles(check_ffmpeg_src, dependencies : libavfilter_dep, name : 'whether libav is provided by FFmpeg')
        gst_libav = ['gst-libav']
    endif
endif

if gst_libav.length() == 0
    message('WARNING: gst-libav not built as ffmpeg >= n3.1.2 not found on the system')
endif

if get_option('enable_python')
    subprojects += ['gst-python']
endif

if not get_option('disable_gst_plugins_ugly')
    subprojects += ['gst-plugins-ugly']
endif

if not get_option('disable_gst_plugins_bad')
    subprojects += ['gst-plugins-bad']
endif

if not get_option('disable_gst_libav')
    subprojects += gst_libav
endif

if not get_option('disable_gst_omx')
    subprojects += ['gst-omx']
endif

if not get_option('disable_gstreamer_vaapi')
  if dependency('libva', version: ['>= 0.30.4', '< 0.99.0'], required : false).found()
    subprojects += ['gstreamer-vaapi']
  else
    message('WARNING: not building gstreamer-vaapi module, missing libva')
  endif
endif

if not get_option('disable_gst_devtools')
  if dependency('json-glib-1.0', required : false).found()
    subprojects += ['gst-devtools']
  else
    message('WARNING: not building gst-devtools module, missing glib-json-1.0')
  endif
endif

if not get_option('disable_gst_editing_services')
  if dependency('libxml-2.0').found()
    subprojects += ['gst-editing-services']
  else
    message('WARNING: not building gst-editing-services module, missing libxml-2.0')
  endif
endif

if not get_option('disable_rtsp_server')
    subprojects += ['gst-rtsp-server']
endif

if not get_option('disable_gstreamer_sharp')
  if add_languages('cs', required : false)
    if meson.version().version_compare('>=0.43')
      subprojects += ['gstreamer-sharp']
    else
      message('WARNING: Not building gstramer-sharp as meson >=0.43 not found.')
    endif
  else
    message('WARNING: Not building gstramer-sharp as no CS compiler found.')
  endif
endif

python3 = find_program('python3')
symlink = '''
import os

os.symlink(os.path.join('@1@', 'subprojects', '@0@'),
    os.path.join('@1@', '@0@'))
'''

foreach custom_subproj: get_option('custom_subprojects').split(',')
    if custom_subproj != ''
        message ('Adding custom subproject ' + custom_subproj)
        subprojects += [custom_subproj]
    endif
endforeach

message('Building subprojects: ' + ', '.join(subprojects))
foreach subproj: subprojects
    default_options = []
    if get_option('disable_introspection')
        default_options += ['disable_introspection=true']
    endif
    subproject(subproj, version: gst_version, default_options: default_options)

    cmdres = run_command(python3, '-c', symlink.format(subproj, meson.current_source_dir()))
    if cmdres.returncode() == 0
       message('Created symlink to ' + subproj)
    else
       message('Could not create symlink to @0@'.format(subproj))
    endif
endforeach

setenv = find_program('gst-uninstalled.py')
run_target('uninstalled', command : [setenv, '--builddir=@0@'.format(meson.build_root()),
	   '--srcdir=@0@'.format(meson.source_root()),
           '--gst-version=@0@'.format(gst_branch)])

update = find_program('git-update')
run_target('git-update', command : [update])
run_target('update', command : [update,
    '--builddir=@0@'.format(meson.current_build_dir())])
