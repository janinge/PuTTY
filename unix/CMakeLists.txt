set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

add_sources_from_current_dir(utils
  utils/arm_arch_queries.c
  utils/block_signal.c
  utils/cloexec.c
  utils/dputs.c
  utils/filename.c
  utils/fontspec.c
  utils/getticks.c
  utils/get_username.c
  utils/keysym_to_unicode.c
  utils/make_dir_and_check_ours.c
  utils/make_dir_path.c
  utils/nonblock.c
  utils/open_for_write_would_lose_data.c
  utils/pgp_fingerprints.c
  utils/pollwrap.c
  utils/signal.c
  utils/x11_ignore_error.c
  # Compiled icon pixmap files
  putty-xpm.c
  putty-config-xpm.c
  pterm-xpm.c
  pterm-config-xpm.c
  # We want the ISO C implementation of ltime(), because we don't have
  # a local better alternative
  ../utils/ltime.c)
add_sources_from_current_dir(eventloop
  cliloop.c uxsel.c)
add_sources_from_current_dir(console
  console.c)
add_sources_from_current_dir(settings
  storage.c)
add_sources_from_current_dir(network
  network.c fd-socket.c agent-socket.c peerinfo.c local-proxy.c x11.c)
add_sources_from_current_dir(sshcommon
  noise.c)
add_sources_from_current_dir(sshclient
  gss.c agent-client.c sharing.c)
add_sources_from_current_dir(sshserver
  sftpserver.c procnet.c)
add_sources_from_current_dir(sftpclient
  sftp.c)
add_sources_from_current_dir(otherbackends
  serial.c)
add_sources_from_current_dir(agent
  agent-client.c)

add_executable(fuzzterm
  ${CMAKE_SOURCE_DIR}/fuzzterm.c
  ${CMAKE_SOURCE_DIR}/be_none.c
  ${CMAKE_SOURCE_DIR}/logging.c
  ${CMAKE_SOURCE_DIR}/noprint.c
  unicode.c
  no-gtk.c)
add_dependencies(fuzzterm generated_licence_h)
target_link_libraries(fuzzterm
  guiterminal eventloop charset settings utils)

add_executable(osxlaunch
  osxlaunch.c)

add_sources_from_current_dir(plink no-gtk.c)
add_sources_from_current_dir(pscp no-gtk.c)
add_sources_from_current_dir(psftp no-gtk.c)
add_sources_from_current_dir(psocks no-gtk.c)

add_executable(psusan
  psusan.c
  ${CMAKE_SOURCE_DIR}/be_none.c
  ${CMAKE_SOURCE_DIR}/nogss.c
  ${CMAKE_SOURCE_DIR}/ssh/scpserver.c
  no-gtk.c
  pty.c)
target_link_libraries(psusan
  eventloop sshserver keygen settings network crypto utils)
installed_program(psusan)

add_library(puttygen-common OBJECT
  ${CMAKE_SOURCE_DIR}/notiming.c
  keygen-noise.c
  no-gtk.c
  noise.c
  storage.c
  ${CMAKE_SOURCE_DIR}/sshpubk.c
  ${CMAKE_SOURCE_DIR}/sshrand.c)

add_executable(puttygen
  ${CMAKE_SOURCE_DIR}/cmdgen.c)
target_link_libraries(puttygen
  puttygen-common keygen console crypto utils)
installed_program(puttygen)

add_executable(cgtest
  ${CMAKE_SOURCE_DIR}/cgtest.c)
target_link_libraries(cgtest
  puttygen-common keygen console crypto utils)

add_executable(testsc
  ${CMAKE_SOURCE_DIR}/testsc.c)
target_link_libraries(testsc keygen crypto utils)

add_executable(testzlib
  ${CMAKE_SOURCE_DIR}/testzlib.c
  ${CMAKE_SOURCE_DIR}/ssh/zlib.c)
target_link_libraries(testzlib utils)

add_executable(uppity
  uppity.c
  ${CMAKE_SOURCE_DIR}/be_none.c
  ${CMAKE_SOURCE_DIR}/ssh/scpserver.c
  no-gtk.c
  pty.c
  ${CMAKE_SOURCE_DIR}/nogss.c)
target_link_libraries(uppity
  eventloop sshserver keygen settings network crypto utils)

if(GTK_FOUND)
  add_sources_from_current_dir(utils
    utils/align_label_left.c
    utils/buildinfo_gtk_version.c
    utils/get_label_text_dimensions.c
    utils/get_x11_display.c
    utils/our_dialog.c
    utils/string_width.c
    columns.c)
  add_sources_from_current_dir(guiterminal
    window.c unifont.c dialog.c config-gtk.c gtk-common.c config-unix.c unicode.c printing.c)
  add_dependencies(guiterminal generated_licence_h) # dialog.c uses licence.h

  add_executable(pageant
    pageant.c
    ${CMAKE_SOURCE_DIR}/be_misc.c
    ${CMAKE_SOURCE_DIR}/be_none.c
    ${CMAKE_SOURCE_DIR}/nogss.c
    askpass.c
    x11.c
    noise.c
    ${CMAKE_SOURCE_DIR}/ssh/x11fwd.c
    ${CMAKE_SOURCE_DIR}/nosshproxy.c)
  target_link_libraries(pageant
    eventloop console agent settings network crypto utils
    ${GTK_LIBRARIES})
  installed_program(pageant)

  add_executable(pterm
    pterm.c
    main-gtk-simple.c
    ${CMAKE_SOURCE_DIR}/be_none.c
    ${CMAKE_SOURCE_DIR}/nogss.c
    ${CMAKE_SOURCE_DIR}/nosshproxy.c
    pty.c)
  target_link_libraries(pterm
    guiterminal eventloop settings charset utils
    ${GTK_LIBRARIES} ${X11_LIBRARIES})
  installed_program(pterm)

  add_executable(ptermapp
    pterm.c
    main-gtk-application.c
    ${CMAKE_SOURCE_DIR}/nocmdline.c
    ${CMAKE_SOURCE_DIR}/be_none.c
    ${CMAKE_SOURCE_DIR}/nogss.c
    ${CMAKE_SOURCE_DIR}/nosshproxy.c
    pty.c)
  target_link_libraries(ptermapp
    guiterminal eventloop settings charset utils
    ${GTK_LIBRARIES} ${X11_LIBRARIES})

  add_executable(putty
    putty.c
    main-gtk-simple.c
    ${CMAKE_SOURCE_DIR}/be_all_s.c)
  target_link_libraries(putty
    guiterminal eventloop sshclient otherbackends settings
    network crypto charset utils
    ${GTK_LIBRARIES} ${X11_LIBRARIES})
  set_target_properties(putty
    PROPERTIES LINK_INTERFACE_MULTIPLICITY 2)
  installed_program(putty)

  add_executable(puttyapp
    putty.c
    main-gtk-application.c
    ${CMAKE_SOURCE_DIR}/nocmdline.c
    ${CMAKE_SOURCE_DIR}/be_all_s.c)
  target_link_libraries(puttyapp
    guiterminal eventloop sshclient otherbackends settings
    network crypto charset utils
    ${GTK_LIBRARIES} ${X11_LIBRARIES})

  add_executable(puttytel
    putty.c
    main-gtk-simple.c
    ${CMAKE_SOURCE_DIR}/be_nos_s.c
    ${CMAKE_SOURCE_DIR}/nogss.c
    ${CMAKE_SOURCE_DIR}/norand.c
    ${CMAKE_SOURCE_DIR}/nocproxy.c
    ${CMAKE_SOURCE_DIR}/nosshproxy.c)
  target_link_libraries(puttytel
    guiterminal eventloop otherbackends settings network charset utils
    ${GTK_LIBRARIES} ${X11_LIBRARIES})
endif()
