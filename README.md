# test-py-flow
Automated deployment pipeline for a 'Hello world' Python app using GitHub Actions.
import http.client
import os
import concurrent.futures
import argparse

def check_directory(host, directory):
    try:
        conn = http.client.HTTPConnection(host)
        conn.request("GET", f"/{directory}/")
        response = conn.getresponse()
        if response.status == 200:
            print(f"Directory found: /{directory}")
        elif response.status == 403:
            print(f"Forbidden: /{directory} exists but is not accessible")
        else:
            print(f"Directory not found: /{directory}")
        conn.close()
    except Exception as e:
        print(f"Error checking /{directory}: {e}")

def brute_force_directories(host, wordlist_path, max_workers):
    if not os.path.exists(wordlist_path):
        print(f"Wordlist file {wordlist_path} not found.")
        return

    with open(wordlist_path, 'r') as wordlist:
        directories = [line.strip() for line in wordlist if line.strip()]

    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = [executor.submit(check_directory, host, directory) for directory in directories]
        for future in concurrent.futures.as_completed(futures):
            future.result()  # Wait for all threads to complete

def main():
    parser = argparse.ArgumentParser(description='Directory brute-forcing tool.')
    parser.add_argument('-H', '--host', required=True, help='Target host (e.g., example.com)')
    parser.add_argument('-w', '--wordlist', required=True, help='Path to the wordlist file')
    parser.add_argument('-t', '--threads', type=int, default=10, help='Number of concurrent threads (default: 10)')
    args = parser.parse_args()

    brute_force_directories(args.host, args.wordlist, args.threads)

if __name__ == "__main__":
    main()
