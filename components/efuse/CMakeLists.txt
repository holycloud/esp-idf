idf_build_get_property(soc_name IDF_TARGET)

if(EXISTS "${COMPONENT_DIR}/${soc_name}")
    include(${COMPONENT_DIR}/${soc_name}/sources.cmake)
    spaces2list(EFUSE_SOC_SRCS)
    add_prefix(COMPONENT_SRCS "${soc_name}/" ${EFUSE_SOC_SRCS})
    set(COMPONENT_ADD_INCLUDEDIRS include ${soc_name}/include)
endif()

list(APPEND COMPONENT_SRCS "src/esp_efuse_api.c"
                           "src/esp_efuse_fields.c"
                           "src/esp_efuse_utility.c")

set(COMPONENT_PRIV_REQUIRES bootloader_support soc)
register_component()

set(GEN_EFUSE_TABLE_ARG --max_blk_len ${CONFIG_EFUSE_MAX_BLK_LEN})

idf_build_get_property(python PYTHON)

###################
# Make common files esp_efuse_table.c and include/esp_efuse_table.h files.
set(EFUSE_COMMON_TABLE_CSV_PATH "${COMPONENT_DIR}/${soc_name}/esp_efuse_table.csv")

add_custom_target(efuse_common_table COMMAND "${python}" "${CMAKE_CURRENT_SOURCE_DIR}/efuse_table_gen.py" ${EFUSE_COMMON_TABLE_CSV_PATH} ${GEN_EFUSE_TABLE_ARG})

###################
# Make custom files project/main/esp_efuse_custom_table.c and project/main/include/esp_efuse_custom_table.h files.
# Path to CSV file is relative to project path for custom CSV files.
if(${CONFIG_EFUSE_CUSTOM_TABLE})
    # Custom filename expands any path relative to the project
    idf_build_get_property(project_dir PROJECT_DIR)
    get_filename_component(EFUSE_CUSTOM_TABLE_CSV_PATH "${CONFIG_EFUSE_CUSTOM_TABLE_FILENAME}" ABSOLUTE BASE_DIR "${project_dir}")
	add_custom_target(efuse_custom_table COMMAND "${python}" "${CMAKE_CURRENT_SOURCE_DIR}/efuse_table_gen.py" ${EFUSE_COMMON_TABLE_CSV_PATH} ${EFUSE_CUSTOM_TABLE_CSV_PATH} ${GEN_EFUSE_TABLE_ARG})
else()
	add_custom_target(efuse_custom_table COMMAND)
endif()#if(${CONFIG_EFUSE_CUSTOM_TABLE})

add_custom_target(show_efuse_table COMMAND "${python}" "${CMAKE_CURRENT_SOURCE_DIR}/efuse_table_gen.py" ${EFUSE_COMMON_TABLE_CSV_PATH} ${EFUSE_CUSTOM_TABLE_CSV_PATH} ${GEN_EFUSE_TABLE_ARG} "--info")

###################
# Generates files for unit test. This command is run manually.
set(EFUSE_TEST_TABLE_CSV_PATH "${COMPONENT_DIR}/test/esp_efuse_test_table.csv")
add_custom_target(efuse_test_table COMMAND "${python}" "${CMAKE_CURRENT_SOURCE_DIR}/efuse_table_gen.py" ${EFUSE_TEST_TABLE_CSV_PATH} ${GEN_EFUSE_TABLE_ARG})
