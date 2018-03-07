#!python
# coding=utf-8
import os
import sys
import requests
from thredds_crawler.crawl import Crawl

import logging
import logging.handlers


class ThreddsIsoHarvester:
    def __init__(self, catalog_url, out_dir, log_file=None, select=None, skip=None, clean=True):

        self.logger = logging.getLogger('thredds_crawler')
        self.logger.setLevel(logging.DEBUG)
        self.logger.handlers = []
        self.__add_stream_logger()

        if log_file is not None:
            self.__add_file_logger(log_file)
        if skip is None:
            skip = Crawl.SKIPS
        else:
            skip.extend(Crawl.SKIPS)

        if not os.path.exists(out_dir):
            os.makedirs(out_dir)

        found_isos = []
        catalog = Crawl(catalog_url, select=select, skip=skip)
        isos = [(d.id, s.get("url")) for d in catalog.datasets for s in d.services if s.get("service").lower() == "iso"]
        for iso in isos:
            try:
                filename = iso[0].replace("/", "_") + ".iso.xml"
                found_isos.append(filename)
                filepath = os.path.join(out_dir, filename)
                self.logger.info("Downloading/Saving %s" % filepath)

                r = requests.get(iso[1], stream=True)
                if r.ok:
                    with open(filepath, 'wb') as f:
                        for chunk in r.iter_content():
                            if chunk:
                                f.write(chunk)
                else:
                    self.logger.info("Got a non-200 status code (%s) from %s" % (r.status_code, iso[1]))
            except KeyboardInterrupt:
                self.logger.info("Caught interrupt, exiting")
                sys.exit(0)
            except BaseException:
                self.logger.exception("Error!")
        if clean:
            self.__clean_not_found_files(out_dir, found_isos)

    def __add_stream_logger(self):
        formatter = logging.Formatter('%(asctime)s - [%(levelname)s] %(message)s')
        ch = logging.StreamHandler()
        ch.setLevel(logging.DEBUG)
        ch.setFormatter(formatter)
        self.logger.addHandler(ch)

    def __add_file_logger(self, log_file):
        formatter = logging.Formatter('%(asctime)s - [%(levelname)s] %(message)s')
        fh = logging.handlers.RotatingFileHandler(
            log_file,
            maxBytes=1024 * 1024 * 10,
            backupCount=5
        )
        fh.setLevel(logging.INFO)
        fh.setFormatter(formatter)
        self.logger.addHandler(fh)

    def __clean_not_found_files(self, check_dir, found_files):
        for f in os.listdir(check_dir):
            if (f not in found_files):
                self.logger.info("%s not found in catalog, deleting...", f)
                os.remove(os.path.join(check_dir, f))
