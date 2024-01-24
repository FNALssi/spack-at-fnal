include(CetGenerateSphinxDocs)

# Are we going with standard version/release, or using git branch/commit
# info?
function(_set_version_release)
  if (${CETMODULES_CURRENT_PROJECT_NAME}_SPHINX_DOC_PUBLISH_FORCE_NUMERIC
      OR NOT EXISTS ${PROJECT_SOURCE_DIR}/.git)
    set(PROJ_VERSION ${CETMODULES_CURRENT_PROJECT_VERSION_MAJOR}.${CETMODULES_CURRENT_PROJECT_VERSION_MINOR})
    set(PROJ_RELEASE ${CETMODULES_CURRENT_PROJECT_VERSION})
  else()
    if (${CETMODULES_CURRENT_PROJECT_NAME}_SPHINX_DOC_PUBLISH_VERSION_BRANCH)
      set(current_branch "${${CETMODULES_CURRENT_PROJECT_NAME}_SPHINX_DOC_PUBLISH_VERSION_BRANCH}")
    else()
      # Try to get the current branch for the version.
      execute_process(COMMAND git branch --show-current
        COMMAND_ERROR_IS_FATAL ANY
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
        OUTPUT_VARIABLE current_branch
        OUTPUT_STRIP_TRAILING_WHITESPACE)
    endif()
    set(PROJ_VERSION "git-${current_branch}")
    # Try to get the current commit for the release.
    execute_process(COMMAND git rev-parse --short HEAD
      COMMAND_ERROR_IS_FATAL ANY
      WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
      OUTPUT_VARIABLE current_commit
      OUTPUT_STRIP_TRAILING_WHITESPACE)
    set(PROJ_RELEASE "${PROJ_VERSION}-${current_commit}")
  endif()
  set(PROJ_VERSION "${PROJ_VERSION}" PARENT_SCOPE)
  set(PROJ_RELEASE "${PROJ_RELEASE}" PARENT_SCOPE)
  return()
endfunction()

# Set Sphinx verbosity-related options based on current CMake reporting
# level.
set(verbose_levels VERBOSE DEBUG TRACE)
set(quiet_levels ERROR WARNING NOTICE STATUS)
if (NOT CMAKE_MESSAGE_LOG_LEVEL OR CMAKE_MESSAGE_LOG_LEVEL IN_LIST quiet_levels)
  set(qv_arg QUIET)
elseif (CMAKE_MESSAGE_LOG_LEVEL IN_LIST verbose_levels)
  list(FIND verbose_levels ${CMAKE_MESSAGE_LOG_LEVEL} default_verbosity)
  math(EXPR verbosity ${default_verbosity} + 1)
  set(qv_arg VERBOSITY ${default_verbosity})
endif()

project_variable(SPHINX_DOC_PUBLISH_ROOT TYPE PATH
  DOCSTRING "destination directory for published HTML documentation"
)

project_variable(SPHINX_DOC_PUBLISH_OLD_RELEASE TYPE BOOL
  DOCSTRING "force overwrite of published documentation for a newer release of the same version"
)

project_variable(SPHINX_DOC_PUBLISH_FORCE_NUMERIC TYPE BOOL
  DOCSTRING "force use of version rather than branch even if we're in a git repository"
)

project_variable(SPHINX_DOC_PUBLISH_VERSION_BRANCH TYPE STRING
  DOCSTRING "specify branch name to be used in PROJ_VERSION (when applicable) instead of using git branch --show-current"
)

project_variable(SPHINX_DOC_EXTRA_ARGS TYPE STRING
  DOCSTRING "extra arguments to sphinx-build")

# Define information for substitution in conf.py.in.
set(PROJ_AUTHORS "scisoft-team <scisoft-team@fnal.gov>")
set(PROJ_NAME "${PROJECT_NAME}")
set(PROJ_COPYRIGHT_START 2024)
set(PROJ_COPYRIGHT_OWNER "Fermi Research Alliance, LLC")

_set_version_release()

configure_file("conf.py.in" "conf.py" @ONLY)

set(doc_root "${CMAKE_CURRENT_SOURCE_DIR}")

set(common_doc_args
  ${qv_arg}
  NITPICKY
  OUTPUT_FORMATS html
  SOURCE_DIR "${doc_root}"
  CONF_DIR "${CMAKE_CURRENT_BINARY_DIR}"
  EXTRA_ARGS ${${CETMODULES_CURRENT_PROJECT_NAME}_SPHINX_DOC_EXTRA_ARGS}
)

if (${CETMODULES_CURRENT_PROJECT_NAME}_SPHINX_DOC_PUBLISH_ROOT)
  cet_publish_sphinx_html(
    "${${CETMODULES_CURRENT_PROJECT_NAME}_SPHINX_DOC_PUBLISH_ROOT}"
    ${PROJ_VERSION}
    ${PUBLISH_OLD_RELEASE}
    ${common_doc_args}
  )
else()
  cet_generate_sphinx_docs(${common_doc_args})
endif()
