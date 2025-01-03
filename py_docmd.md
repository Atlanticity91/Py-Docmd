#!/usr/bin/env python3

# --- INCLUDES
import argparse
import pathlib
import os
import re
import shutil

from types import SimpleNamespace

# --- MARKDOWN WRITING
def parse_param( line ) :
    print( line.split( ) )

    return [ ]

def parse_note( line ) :
    print( line.split( ) )

    return [ ]

def parse_return( line ) :
    print( line.split( ) )

    return [ ]

# --- GLOBAL TABLE
TAG_TABLE = [
    ( '@param', r'(\S+) : (.*)', parse_param ),
    ( '@note', r': (.*)', parse_note ),
    ( '@return', r': (.*)', parse_return )
]

# --- CORE LOGIC
def create_directory( directory ) :
    if os.path.isdir( directory ) :
        shutil.rmtree( directory )

    os.mkdir( directory )

def format_name( text ) :
    return f'{text[ 0 ].upper( )}{text[ 1 : ]}'

def format_link( text ) :
    return f'[{format_name( text )}]({text}.md)'

def create_markdown( md_path, index_name, name, documentation ) :
    with open( md_path, 'w' ) as file :
        file.write( f'# {format_link( index_name )} : {format_name( name )}\n---\n\n' )

def parse_comment( comment ) :
    documentation = { }
    lines = [ line.strip( ' * ' ) for line in comment.split( '\n' )[ 1 : ] ]
    lines = [ line.strip( ) for line in lines ]
    name = lines[ 0 ].split( )[ 0 ].lower( )

    for line in lines[ 1 : ] :
        if line.endswith( '/' ) or not line.strip( ) :
            continue

        for tag in TAG_TABLE :
            if not line.startswith( tag[ 0 ] ) :
                continue

            content = tag[ 2 ]( line.replace( tag[ 0 ], '' ) )
            documentation.update( { f'{tag[ 0 ][ 1 : ]}' : content } )

            break

    return name, SimpleNamespace( **documentation )

def create_index_markdown( md_path, name, pages ) :
    with open( md_path, 'w' ) as file :
        file.write( f'# {format_link( name )}\n---\n\n' )

        for page in pages :
            file.write( f' * [{format_name( page )}]({name}_{page}.md)\n' )

def parse_file( md_destination, file_path ) :
    content = None
    name = file_path.split( '/' )[ -1 ]
    name = name.split( '.' )[ -2 ]
    name = name.lower( )
    md_destination = f'{md_destination}{name}/'

    create_directory( md_destination )

    with open( file_path, 'r' ) as file :
        content = file.read( )

    pattern = re.compile( r'/\*\*.*?\*/.*?(?=[;{])', re.DOTALL )
    comments = re.findall( pattern, content )
    pages = [ ]

    for comment in comments :
        page_name, documentation = parse_comment( comment )

        pages.append( page_name )

        create_markdown( f'{md_destination}{name}_{page_name}.md', name, page_name, documentation )

    create_index_markdown( f'{md_destination}{name}.md', name, pages )

def parse_directory( md_destination, directory_path ) :
    if os.path.isdir( directory_path ) :
        for path in os.listdir( directory_path ) :
            path = f'{directory_path}{path}'

            if os.path.isdir( path ) :
                parse_directory( md_destination, path )
            elif os.path.isfile( path ) :
                parse_file( md_destination, path )
    elif os.path.isfile( directory_path ) :
        parse_file( md_destination, directory_path )

# --- MAIN LOGIC 
if __name__ == '__main__' :
    print( 
        "> === Py-Docmd ===\n"
        "> Utility tool for generating MD-Book markdown files from C/C++ headers comments."
    )

    parser = argparse.ArgumentParser( )

    parser.add_argument( '-o', '--out', type=str, nargs='*', help='Set the output mdbook markdown output directory' )
    parser.add_argument( '-f', '--file', required=True, type=str, nargs='*', help='Set mdbook source code file or directory.' )

    arguments = parser.parse_args( )

    md_destination = 'Mdbook/' if arguments.out is None else arguments.out

    create_directory( md_destination )

    for path in arguments.file : 
        parse_directory( md_destination, path )
