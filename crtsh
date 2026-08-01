#!/bin/bash

# Define variables
DOMAIN_TARGET=""
DOMAIN_LIST_FILE=""
OUTPUT_FILE="crtsh_output.txt"
SILENT=false

# --- Function Definitions ---

# Banner function
banner() {
    cat << "EOF"
  ________  ________  _________  ________  ___  ___    
 |\  ____\|\  __  \|\___  ___\\  ____\|\  \|\  \   
 \ \ \___|\ \  \|\  \|___ \  \_\ \ \___|\ \  \\\  \  
  \ \ \    \ \  \ _  _\   \ \  \ \ \_____  \ \  \ __  \ 
   \ \ \____\ \  \\  \|   \ \  \ \|____|\  \ \  \ \  \ 
    \ \_______\ \__\\ _\    \ \__\  ____\_\  \ \__\ \__\
    \|_______|\|__|\|__|    \|__| |\_________\|__|\|__|
                                 \|_________|          
     Subdomain Recon Tool by 0xmun1r (crt.sh based, optimized by volkansah)
EOF
}

# Help function (using $0 for script name)
usage() {
    echo "Usage: $(basename "$0") [options]"
    echo ""
    echo "Options:"
    echo "  -d <domain>        Enumerate subdomains for a single domain"
    echo "  -l <file>          Enumerate subdomains for a list of domains (one per line)"
    echo "  -o <output.txt>    Save output to specified file (default: $OUTPUT_FILE)"
    echo "  -s                 Silent mode (no banner or terminal output)"
    echo "  -h                 Show this help message and exit"
    echo ""
    echo "Examples:"
    echo "  $(basename "$0") -d example.com"
    echo "  $(basename "$0") -l domains.txt -o results.txt"
    exit 0
}

# Function to fetch and process subdomains (optimized, no jq)
fetch_subdomains() {
    local dom="$1"
    local outfile="$2"
    
    [ "$SILENT" = false ] && echo "[*] Enumerating subdomains for: $dom"
    
    # Core logic: Fetch JSON, extract name_value, clean, filter, and normalize
    local subs=$(curl -s "https://crt.sh/?q=%25.$dom&output=json" | \
        grep -oP '"name_value":"\K[^"]+' | \
        sed -E 's/\*\.//g' | \
        tr '[:upper:]' '[:lower:]' | \
        tr ',' '\n' | \
        grep -E "^([a-z0-9.-]+\.)*$dom$" | \
        grep -v -E "(^$dom$)|(@)" | \
        sort -u)
    
    local count=$(echo "$subs" | grep -c .)

    if [ "$count" -gt 0 ]; then
        if [ "$SILENT" = false ]; then
            echo "[+] Found $count subdomains for $dom"
            echo "$subs"
        fi
        
        # Append to output file
        echo -e "\n# Subdomains for $dom ($count found)" >> "$outfile"
        echo "$subs" >> "$outfile"
    else
        [ "$SILENT" = false ] && echo "[-] No subdomains found for $dom or an error occurred."
    fi
}

# --- Argument Parsing (using getopts) ---
while getopts "d:l:o:sh" opt; do
    case ${opt} in
        d) DOMAIN_TARGET="$OPTARG" ;;
        l) DOMAIN_LIST_FILE="$OPTARG" ;;
        o) OUTPUT_FILE="$OPTARG" ;;
        s) SILENT=true ;;
        h) usage ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            usage
            ;;
        :)
            echo "Option -$OPTARG requires an argument." >&2
            usage
            ;;
    esac
done
shift $((OPTIND -1)) # Consume the parsed arguments

# --- Main Logic ---

# Show banner if not in silent mode
if [ "$SILENT" = false ]; then
    banner
fi

# Clear output file only if a target is provided
if [[ -n "$DOMAIN_TARGET" || -n "$DOMAIN_LIST_FILE" ]]; then
    [ "$SILENT" = false ] && echo "[*] Clearing previous content in: $OUTPUT_FILE"
    > "$OUTPUT_FILE"
fi

if [[ -n "$DOMAIN_TARGET" ]]; then
    # Single Domain Mode
    fetch_subdomains "$DOMAIN_TARGET" "$OUTPUT_FILE"
    
elif [[ -n "$DOMAIN_LIST_FILE" ]]; then
    # Domain List Mode
    if [[ ! -f "$DOMAIN_LIST_FILE" ]]; then
        echo "[-] Error: Domain list file not found: $DOMAIN_LIST_FILE" >&2
        exit 1
    fi
    
    while IFS= read -r dom; do
        # Skip empty lines or lines starting with a comment (#)
        [[ -z "$dom" || "$dom" =~ ^# ]] && continue
        fetch_subdomains "$dom" "$OUTPUT_FILE"
    done < "$DOMAIN_LIST_FILE"
    
else
    # No input provided
    echo "[-] Error: No domain (-d) or domain list (-l) provided."
    usage
fi

[ "$SILENT" = false ] && echo "[+] All results saved to: $OUTPUT_FILE"
