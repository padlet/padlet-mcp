<p align="center">
  <img src="https://padlet.net/padlet-intelligence/padlet_crane_outlined.png" alt="Padlet" width="120">
</p>

# Padlet MCP Server

Work with your [Padlet](https://padlet.com) boards from GitHub Copilot and any other AI agent
that speaks the [Model Context Protocol](https://modelcontextprotocol.io) (MCP).

Padlet is a visual board for collecting and sharing ideas, used by teachers, students and teams.
A board (a "padlet") holds posts with text, links, images, video and files, grouped into
sections. This server lets an agent find, read, create and edit your padlets for you, in plain
language.

- **Hosted by Padlet.** Nothing to install or run. The server is at `https://mcp.padlet.com/`
  (streamable HTTP).
- **No API key.** You sign in with your own Padlet account in the browser the first time your
  client connects (OAuth).
- **Acts as you.** The agent can see and change only what you can see and change in Padlet.

This repository holds documentation only. The server itself is run by Padlet; its source code
is not published here.

## Contents

- [Set up in VS Code with GitHub Copilot](#set-up-in-vs-code-with-github-copilot)
- [Copilot cloud agent (coding agent)](#copilot-cloud-agent-coding-agent)
- [Any other MCP client](#any-other-mcp-client)
- [Signing in](#signing-in)
- [What you can ask for](#what-you-can-ask-for)
- [Toolsets](#toolsets)
- [Limits and safety](#limits-and-safety)
- [Support, privacy and terms](#support-privacy-and-terms)

## Set up in VS Code with GitHub Copilot

**One click:**

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Padlet-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=padlet&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.padlet.com%2F%22%7D)
[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install_Padlet-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=padlet&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fmcp.padlet.com%2F%22%7D&quality=insiders)

**By hand:** run **MCP: Open User Configuration** from the Command Palette (or create
`.vscode/mcp.json` to share the setup with everyone working in a repository) and add:

```json
{
  "servers": {
    "padlet": {
      "type": "http",
      "url": "https://mcp.padlet.com/"
    }
  }
}
```

Or from a terminal:

```bash
code --add-mcp '{"name":"padlet","type":"http","url":"https://mcp.padlet.com/"}'
```

Then open Copilot Chat in **Agent** mode. When the server starts, VS Code asks you to sign in to
Padlet; see [Signing in](#signing-in).

## Copilot cloud agent (coding agent)

Not supported at the moment. GitHub's
[documentation](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/extend-coding-agent-with-mcp)
says the Copilot cloud agent and Copilot code review "do not currently support remote MCP
servers that leverage OAuth for authentication and authorization". Padlet's server signs you in
with OAuth only and has no API-key option, so it can't be added there. Use Copilot Chat in VS
Code instead.

## Any other MCP client

Any client that supports remote MCP servers over streamable HTTP with OAuth can connect. Most
clients take a config like this:

```json
{
  "mcpServers": {
    "padlet": {
      "type": "http",
      "url": "https://mcp.padlet.com/"
    }
  }
}
```

The field names vary a little between clients (`servers` or `mcpServers`, `http` or
`streamable-http`), so check your client's documentation. The URL is always
`https://mcp.padlet.com/`.

## Signing in

1. The first time your client connects, it registers itself with Padlet and opens a Padlet
   sign-in page in your browser.
2. Sign in with your Padlet account if you aren't already, and approve the request.
3. Go back to your client. You're connected.

Your client keeps the sign-in, so you won't be asked again each time. Every tool call runs as
the signed-in user and follows Padlet's normal permissions: the agent can open the padlets you
can open and change the ones you can change, nothing more.

## What you can ask for

For teaching and classroom work:

- "Make a padlet called *Persuasive Writing: Week 3* with sections for Ideas, Drafts and
  Feedback."
- "Read my *Exit Tickets* padlet and summarise what students found hardest."
- "Comment on each post in the Drafts section with one strength and one next step."
- "Freeze my *Unit 2 Reflections* padlet so no one can post any more."

For teams and projects:

- "Find my board about the Q3 retro and group the posts by theme."
- "Add each item in TODO.md as a post on my *Sprint Planning* board."
- "Export the *Design Feedback* padlet as a PDF."
- "Change the *Launch Plan* padlet to a timeline layout with a dark wallpaper."

The agent replies with links to the padlets it worked on, so you can open them in Padlet.

## Toolsets

The server groups its tools into toolsets. With the plain URL `https://mcp.padlet.com/` you get
every toolset available to your account:

| Toolset | Tools | What it covers |
| --- | --- | --- |
| `core` | 17 + 2 | Find, read and export padlets; create padlets; create, edit, pin, connect and restore posts; edit sections; add and delete comments; analyse attachments. Also brings the 2 file-upload tools for attaching a file from your computer. |
| `utility` | 1 | Current date and time (`get_time`) |
| `board_admin` | 29 | Per-padlet settings: appearance, layout, wallpaper, post fields and sorting, access and collaborators, submission forms, breakout links, freezing, remaking, automations, Arcade activities, grading and grade sync to your LMS, whiteboard tools |
| `library` | 24 | Padlet for Schools and Teams library admin: members and invites, onboarding, Google/Microsoft sign-in and SAML SSO, approved domains, permissions, content safety, allowed attachment types |

So the plain URL serves **49 tools**, or **73** if you are an owner, admin or teacher in a
Padlet for Schools or Teams library. The `library` toolset appears only for those accounts.

Some clients limit how many tools a model can see across all connected servers. To load only part
of Padlet, add `?toolsets=` with a comma-separated list:

```
https://mcp.padlet.com/?toolsets=core
https://mcp.padlet.com/?toolsets=core,board_admin
```

`?toolsets=core` gives 19 tools (the 17 core tools plus the 2 upload tools). Unknown toolset
names are ignored.

For a strictly read-only connection, use `https://mcp.padlet.com/read-only`. It serves 5 tools
that only find and read padlets (`get_current_user_info`, `list_padlets`, `search_walls`,
`get_padlet_content`, `export_padlet`) and can't create or change anything.

<details>
<summary>All tool names by toolset</summary>

**core:** `get_current_user_info`, `list_padlets`, `search_walls`, `get_padlet_content`,
`export_padlet`, `create_padlet`, `update_padlet_title_description`, `create_posts`,
`update_posts`, `update_section`, `pin_post`, `create_post_connection`,
`delete_post_connection`, `restore_posts_and_sections`, `add_comment_to_post`,
`delete_comment`, `analyze_post_attachments`. Upload tools served with core:
`start_attachment_upload`, `complete_attachment_upload`.

**utility:** `get_time`

**board_admin:** `update_padlet_icon`, `update_padlet_appearance`, `update_padlet_layout`,
`update_padlet_wallpaper`, `update_padlet_map_style`, `get_padlet_post_properties`,
`update_padlet_post_properties`, `update_padlet_post_sort`, `update_padlet_settings`,
`get_padlet_content_settings`, `update_padlet_content_settings`, `get_padlet_access_settings`,
`update_padlet_access_settings`, `invite_email_as_padlet_collaborator`,
`update_delete_padlet_collaborator_user`, `update_submission_form_settings`,
`update_breakout_settings`, `delete_custom_fields`, `freeze_padlet`, `create_automation`,
`delete_automations`, `get_padlet_automations`, `remake_padlet`, `update_auto_remake_settings`,
`create_padlet_arcade_activity`, `grade_posts`, `sync_grades_to_lms`,
`get_whiteboard_tool_settings`, `update_whiteboard_tool_settings`

**library:** `list_libraries`, `get_library_info`, `get_library_invite_links`,
`update_library_invite_links`, `send_library_email_invites`, `remove_library_email_invites`,
`bulk_invite_library_members`, `get_library_onboarding_flow`,
`complete_library_onboarding_step`, `get_library_oauth_settings`,
`update_library_oauth_settings`, `get_library_saml_settings`, `create_library_saml_connection`,
`update_library_saml_connection`, `validate_saml_metadata_url`,
`get_library_approved_domains`, `add_library_approved_domain`,
`remove_library_approved_domains`, `get_library_permissions`, `update_library_permission`,
`get_library_content_safety`, `update_library_content_safety`,
`get_library_attachment_type_settings`, `update_library_attachment_type`

</details>

## Limits and safety

- **Everything runs as you.** There is no shared service account. Library tools still check
  your role in that library on every call.
- **Changes you can undo, and changes you can't.** Deleted posts and sections can be restored
  with `restore_posts_and_sections`. Tools that delete or overwrite something are marked as
  destructive, so clients that honour MCP tool annotations can ask you before running them.
- **Sandbox (whiteboard) padlets** are only partly supported; the other padlet formats are
  fully supported.

## Support, privacy and terms

- Padlet: <https://padlet.com>
- Help centre: <https://padlet.help>
- Contact: [hello@padlet.com](mailto:hello@padlet.com)
- Privacy Policy: <https://padlet.com/about/privacy>
- Terms of Service: <https://padlet.com/about/terms>
- Security: see [SECURITY.md](SECURITY.md)

## License

MIT, see [LICENSE](LICENSE). This repository contains documentation only; the MCP server is
hosted by Padlet at `https://mcp.padlet.com/`.

Padlet and the Padlet logo are trademarks of Wallwisher, Inc. The MIT license covers the text in
this repository; it does not grant any right to use Padlet's name or marks.
