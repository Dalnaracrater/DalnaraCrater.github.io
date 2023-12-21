---
layout: post
title: angr installation
date: 2019-12-28 05:24:00
tags: Programming-language
---
angr is a powerful analysis tool.
you can download it by using this command.
'''
pip install angr
'''

Or just download the docker image.
'''
# install docker
curl -sSL https://get.docker.com/ | sudo sh

# pull the docker image
sudo docker pull angr/angr

# run it
sudo docker run -it angr/angr
'''
