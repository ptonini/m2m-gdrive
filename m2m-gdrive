import sys
from pydrive.drive import GoogleDrive
from pydrive.auth import GoogleAuth


def get_child_obj(obj_type, drive, parent_id, obj_title=None):

    try:
        obj_list = drive.ListFile({'q': '"' + parent_id + '" in parents and trashed=false'}).GetList()
    except Exception as e:
        print e
        sys.exit(1)

    if obj_title:

        matched = list()
        for obj in obj_list:
            if obj['title'] == obj_title:
                if obj['mimeType'] != 'application/vnd.google-apps.folder':
                    matched.append(obj)
                if obj_type == 'folder' and obj['mimeType'] == 'application/vnd.google-apps.folder':
                    matched.append(obj)
    else:
        matched = obj_list

    if len(matched) == 1:
        return matched[0]
    elif len(matched) == 0:
        print('Could not match {} "{}"'.format(obj_type, obj_title))
        sys.exit(1)
    else:
        print('To many matches for "{}" ({})'.format(obj_title, len(matched)))
        sys.exit(1)


def main():
    settings_file = 'settings.yaml'
    http_timeout = 5
    package_path = 'repo/app2/3.0'
    save_to = 'worker-0.0.1-SNAPSHOT-jar-with-dependencies.jar'

    gauth = GoogleAuth(settings_file=settings_file, http_timeout=http_timeout)
    drive = GoogleDrive(gauth)

    path_list = package_path.split('/')

    folder_obj = get_child_obj('folder', drive, 'root', path_list[0])
    path_list.pop(0)

    path_list_depth = len(path_list)
    index = 0

    while path_list_depth > 0:
        folder_obj = get_child_obj('folder', drive, folder_obj['id'], path_list[index])
        path_list_depth -= 1
        index += 1

    file_obj = get_child_obj('file', drive, folder_obj['id'])
    package = drive.CreateFile({'id': file_obj['id'], 'mimeType': file_obj['mimeType']})

    try:
        package.GetContentFile(save_to)
    except Exception as e:
        print e
        sys.exit(1)
    else:
        print file_obj['title']

if __name__ == '__main__':
    main()
