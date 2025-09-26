# Complete List of SNMP-Related Files in Glances Repository

## Core SNMP Implementation Files

### Primary SNMP Files
1. **`glances/snmp.py`** - Core SNMP client implementation using PySNMP
2. **`glances/stats_client_snmp.py`** - SNMP statistics client manager

### Plugin Framework
3. **`glances/plugins/plugin/model.py`** - Base plugin model with SNMP support methods

## Application Integration Files

### Client and Server Logic
4. **`glances/client.py`** - Client connection handling with SNMP fallback
5. **`glances/client_browser.py`** - Browser interface with SNMP status indicators
6. **`glances/main.py`** - Command-line argument parsing for SNMP options

## Plugin Files with SNMP Support

### Plugins with Dedicated SNMP OID Definitions
7. **`glances/plugins/network/__init__.py`** - Network interface monitoring via IF-MIB
8. **`glances/plugins/mem/__init__.py`** - Memory statistics via UCD-SNMP-MIB
9. **`glances/plugins/cpu/__init__.py`** - CPU utilization monitoring  
10. **`glances/plugins/fs/__init__.py`** - Filesystem usage monitoring
11. **`glances/plugins/load/__init__.py`** - System load averages
12. **`glances/plugins/system/__init__.py`** - Basic system information
13. **`glances/plugins/uptime/__init__.py`** - System uptime information

### Plugins with SNMP Input Method Support
14. **`glances/plugins/irq/__init__.py`** - Interrupt statistics
15. **`glances/plugins/ip/__init__.py`** - IP configuration  
16. **`glances/plugins/connections/__init__.py`** - Network connections
17. **`glances/plugins/smart/__init__.py`** - Hard drive SMART data
18. **`glances/plugins/sensors/sensor/glances_batpercent.py`** - Battery sensors
19. **`glances/plugins/sensors/sensor/glances_hddtemp.py`** - HDD temperature
20. **`glances/plugins/sensors/__init__.py`** - Hardware sensors
21. **`glances/plugins/raid/__init__.py`** - RAID arrays status
22. **`glances/plugins/wifi/__init__.py`** - WiFi interface information
23. **`glances/plugins/memswap/__init__.py`** - Memory and swap statistics
24. **`glances/plugins/core/__init__.py`** - Core system metrics
25. **`glances/plugins/percpu/__init__.py`** - Per-CPU statistics
26. **`glances/plugins/ports/__init__.py`** - Port monitoring
27. **`glances/plugins/amps/__init__.py`** - Application monitoring
28. **`glances/plugins/quicklook/__init__.py`** - Quick overview plugin

## UI and Output Files

### User Interface Components
29. **`glances/outputs/glances_curses.py`** - Terminal UI with SNMP status
30. **`glances/outputs/glances_curses_browser.py`** - Browser UI for SNMP servers

## Configuration and Documentation Files

### Project Configuration
31. **`pyproject.toml`** - Project configuration with SNMP dependencies
32. **`optional-requirements.txt`** - Optional dependency specifications
33. **`.coveragerc`** - Code coverage configuration mentioning SNMP

### Documentation
34. **`docs/man/glances.1`** - Manual page with SNMP command-line options
35. **`docs/quickstart.rst`** - Quick start guide with SNMP examples  
36. **`docs/cmds.rst`** - Command reference documentation
37. **`README.rst`** - Main README with SNMP installation instructions
38. **`NEWS.rst`** - Release notes mentioning SNMP features

## Summary Statistics

- **Total SNMP-related files**: 38
- **Core implementation files**: 3  
- **Application integration files**: 3
- **Plugins with SNMP OID definitions**: 7
- **Plugins with SNMP support**: 21
- **UI/output files**: 2
- **Configuration files**: 3
- **Documentation files**: 5

## File Categories by Type

### Python Implementation (28 files)
- Core SNMP library and client code
- Plugin implementations with OID definitions  
- Application integration and UI components

### Configuration (3 files)
- Package dependencies and build configuration
- Code coverage and testing setup

### Documentation (7 files)  
- User manuals and guides
- Installation instructions
- Command reference and examples

This comprehensive file listing demonstrates the extensive SNMP integration throughout the Glances codebase, touching nearly every aspect of the application from core functionality to user interfaces and documentation.